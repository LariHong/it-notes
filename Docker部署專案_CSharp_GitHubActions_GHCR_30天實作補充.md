# Docker 部署專案：C# / ASP.NET Core + GitHub Actions + GHCR 30 天實作補充

## 來源與定位

- 主線來源：30 天學習 Docker 部署你的專案：https://ithelp.ithome.com.tw/m/users/20151035/ironman/6311
- 延伸基準：ASP.NET Core Docker 官方教學：https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/docker/building-net-docker-images
- 延伸基準：Microsoft .NET Docker sample：https://github.com/dotnet/dotnet-docker/tree/main/samples/aspnetapp
- 延伸基準：GitHub Actions 發布 Docker images：https://docs.github.com/en/actions/tutorials/publish-packages/publish-docker-images
- 延伸基準：GitHub Container Registry：https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry

> 這份筆記不修改原本的 Docker 主線筆記，而是把那條「本機 Docker -> image -> registry -> 遠端部署」路線改寫成 C# / ASP.NET Core 與 GitHub Actions / GHCR 版本。小專案只作為可複製範例，不會真的建立到使用者 GitHub repo。

## 這份補充要解決什麼

原系列很適合建立 Docker 部署直覺，但如果你的主力語言是 C#，通常會卡在三件事：ASP.NET Core container port 怎麼設定、Dockerfile 怎麼寫才符合 .NET 專案、GitHub Actions 要怎麼把 image 推到 GitHub Container Registry。這份補充把這三件事串成一條可練習的路線。

## 主線專案

### 專案最終會長成什麼

一個最小 ASP.NET Core Web API，提供 `/health` 與 `/orders/{id}`。本機可以 `dotnet build`，可以用 Dockerfile build 成 image，可以用 GitHub Actions 推到 `ghcr.io`，遠端主機可以 pull image 後用 `docker run` 或 compose 啟動。

### 需要的檔案地圖

| 檔案 | 負責的事情 |
| --- | --- |
| `src/DeployDemo.Api/Program.cs` | 最小 API 入口，提供部署後可驗證的 endpoint |
| `src/DeployDemo.Api/DeployDemo.Api.csproj` | .NET Web SDK 專案檔，讓 restore/build/publish 有入口 |
| `Dockerfile` | multi-stage build，先 publish，再用 ASP.NET runtime image 執行 |
| `docker-compose.yml` | 本機或遠端啟動 API container 的範例 |
| `.github/workflows/docker-publish.yml` | GitHub Actions build 並 push image 到 GHCR |
| GitHub Packages / GHCR | 儲存 workflow 產出的 container image |

### 30 天交付物地圖

| Day | 交付物 |
| --- | --- |
| 1 | 建立 DeployDemo.Api 的學習目標與資料流 |
| 2 | 讓專案有 /health 與 /orders/{id} 可驗證端點 |
| 3 | 確認 C# 專案不是只停在 Docker 層，而是本身可編譯 |
| 4 | 用 SDK image restore/publish 專案 |
| 5 | 用 aspnet runtime image 跑已 publish 的 dll |
| 6 | 讓 host port 與 container port 對齊，不再誤用舊範例 port 80 |
| 7 | 把 C# 專案變成 image |
| 8 | 從 image 建立 container 並驗證 /health |
| 9 | 用 compose 保存本機啟動設定，減少手打指令 |
| 10 | 把 image 命名成 GHCR 可接受格式 |
| 11 | 理解本機 push 需要 PAT，Actions push 可用 GITHUB_TOKEN |
| 12 | 知道 Actions 自動化前，本機推 image 是怎麼運作 |
| 13 | 用 YAML 定義 main push 觸發 build/push |
| 14 | 給 GITHUB_TOKEN packages: write，讓 workflow 可以推 GHCR |
| 15 | 讓 runner 取得 repo 並啟用 Docker Buildx |
| 16 | 用 GitHub Actor 與 GITHUB_TOKEN 登入 ghcr.io |
| 17 | 用 Docker 官方 action build 並 push image |
| 18 | 同時提供易讀 tag 與可追蹤 commit tag |
| 19 | 把 package 關聯回 GitHub repo |
| 20 | 從 workflow log 判斷是 restore、build、login 還是 push 失敗 |
| 21 | 理解 GHCR package 預設可能是 private，要調整 access |
| 22 | 部署主機只需要 pull 已發布 image，不一定要有 source code |
| 23 | 用 docker run 驗證 image 在主機可啟動 |
| 24 | 把 build 改成 image，讓部署端拉取 GHCR 成品 |
| 25 | 分清楚 GitHub Secrets、container env、appsettings |
| 26 | 在 workflow build 後至少檢查 image 可建立或 API 可回應 |
| 27 | 用 sha tag 回到上一版 image |
| 28 | 本任務只把專案作為筆記中的可複製範例 |
| 29 | 把 GitHub Actions 上線前檢查寫成清單 |
| 30 | 把原 Docker 部署主線轉成 C# + GHCR 可實作流程 |

### 主線端到端流程

需求要部署 C# API -> 建立 ASP.NET Core API -> 本機 build -> 寫 Dockerfile -> 本機 build/run image -> 加 compose -> 建 GitHub Actions -> login GHCR -> build/push image -> GHCR package -> 遠端主機 pull/run -> `/health` 驗證。

### 主線做完後檢查

- `dotnet build` 成功，沒有 C# 編譯錯誤。
- `Dockerfile` 使用 build stage 與 runtime stage，runtime image 只帶 published output。
- GitHub Actions workflow 有 `packages: write`、GHCR login、build-push-action、`latest` 與 `${{ github.sha }}` tag。
- 遠端部署說明使用 image tag，不要求在遠端主機 clone source code。
- 筆記中的 secret 全部使用 placeholder，不出現真實 token。

## 小專案完整範例

### `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapGet("/health", () => Results.Ok(new
{
    Status = "ok",
    Service = "DeployDemo.Api",
    Runtime = System.Runtime.InteropServices.RuntimeInformation.FrameworkDescription,
    Time = DateTimeOffset.UtcNow
}));

app.MapGet("/orders/{id:int}", (int id) =>
{
    if (id <= 0)
    {
        return Results.BadRequest(new { Error = "Order id must be greater than 0." });
    }

    return Results.Ok(new
    {
        Id = id,
        Status = "ready-to-ship",
        CheckedAt = DateTimeOffset.UtcNow
    });
});

app.Run();
```

### `Dockerfile`

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY src/DeployDemo.Api/DeployDemo.Api.csproj src/DeployDemo.Api/
RUN dotnet restore src/DeployDemo.Api/DeployDemo.Api.csproj
COPY src/DeployDemo.Api/ src/DeployDemo.Api/
RUN dotnet publish src/DeployDemo.Api/DeployDemo.Api.csproj -c Release -o /app/publish --no-restore

FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS final
WORKDIR /app
COPY --from=build /app/publish .
USER app
EXPOSE 8080
ENTRYPOINT ["dotnet", "DeployDemo.Api.dll"]
```

### `docker-compose.yml`

```yaml
services:
  api:
    image: ghcr.io/YOUR_GITHUB_ACCOUNT/deploy-demo-api:local
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      ASPNETCORE_HTTP_PORTS: "8080"
```

### `.github/workflows/docker-publish.yml`

```yaml
name: Build and publish DeployDemo.Api image

on:
  push:
    branches: [ main ]
  workflow_dispatch:

permissions:
  contents: read
  packages: write

env:
  IMAGE_NAME: ghcr.io/${{ github.repository_owner }}/deploy-demo-api

jobs:
  build-and-push:
    runs-on: ubuntu-latest

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
          file: ./Dockerfile
          push: true
          tags: |
            ${{ env.IMAGE_NAME }}:latest
            ${{ env.IMAGE_NAME }}:${{ github.sha }}
          labels: |
            org.opencontainers.image.source=${{ github.server_url }}/${{ github.repository }}
```

## Day 1 - 把原系列主線改成 C# 部署題目

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「把原系列主線改成 C# 部署題目」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「建立 DeployDemo.Api 的學習目標與資料流」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「把原系列主線改成 C# 部署題目」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：建立 DeployDemo.Api 的學習目標與資料流。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`README.md、src/DeployDemo.Api/Program.cs`。
- 資料與部署流向是：需求 -> ASP.NET Core API -> Dockerfile -> GitHub Actions -> GHCR -> 遠端主機 pull image。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「把原系列主線改成 C# 部署題目」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`README.md、src/DeployDemo.Api/Program.cs`。
- 資料怎麼流：需求 -> ASP.NET Core API -> Dockerfile -> GitHub Actions -> GHCR -> 遠端主機 pull image。
- 流程路線圖：需求 -> 找責任層 -> 修改 `README.md` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
dotnet new web -n DeployDemo.Api -o src/DeployDemo.Api
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「把原系列主線改成 C# 部署題目」。

具體流程：
1. 打開或建立 `README.md`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「把原系列主線改成 C# 部署題目」，不要跨層修太多東西。
2. 修改或檢查 `README.md、src/DeployDemo.Api/Program.cs`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 1 的重點是：把「把原系列主線改成 C# 部署題目」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 2 - 建立 ASP.NET Core 最小 API

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「建立 ASP.NET Core 最小 API」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「讓專案有 /health 與 /orders/{id} 可驗證端點」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「建立 ASP.NET Core 最小 API」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：讓專案有 /health 與 /orders/{id} 可驗證端點。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`src/DeployDemo.Api/Program.cs`。
- 資料與部署流向是：HTTP request -> Minimal API route -> Results.Ok / BadRequest -> JSON response。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「建立 ASP.NET Core 最小 API」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`src/DeployDemo.Api/Program.cs`。
- 資料怎麼流：HTTP request -> Minimal API route -> Results.Ok / BadRequest -> JSON response。
- 流程路線圖：需求 -> 找責任層 -> 修改 `src/DeployDemo.Api/Program.cs` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
dotnet run --project src/DeployDemo.Api/DeployDemo.Api.csproj --urls http://localhost:5000
curl http://localhost:5000/health
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「建立 ASP.NET Core 最小 API」。

具體流程：
1. 打開或建立 `src/DeployDemo.Api/Program.cs`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「建立 ASP.NET Core 最小 API」，不要跨層修太多東西。
2. 修改或檢查 `src/DeployDemo.Api/Program.cs`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 2 的重點是：把「建立 ASP.NET Core 最小 API」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 3 - 本機 restore/build 驗證

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「本機 restore/build 驗證」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「確認 C# 專案不是只停在 Docker 層，而是本身可編譯」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「本機 restore/build 驗證」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：確認 C# 專案不是只停在 Docker 層，而是本身可編譯。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`DeployDemo.Api.csproj、dotnet restore、dotnet build`。
- 資料與部署流向是：csproj -> NuGet restore -> compiler -> bin/Debug/net10.0。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「本機 restore/build 驗證」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`DeployDemo.Api.csproj、dotnet restore、dotnet build`。
- 資料怎麼流：csproj -> NuGet restore -> compiler -> bin/Debug/net10.0。
- 流程路線圖：需求 -> 找責任層 -> 修改 `DeployDemo.Api.csproj` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
dotnet restore src/DeployDemo.Api/DeployDemo.Api.csproj
dotnet build src/DeployDemo.Api/DeployDemo.Api.csproj --no-restore
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「本機 restore/build 驗證」。

具體流程：
1. 打開或建立 `DeployDemo.Api.csproj`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「本機 restore/build 驗證」，不要跨層修太多東西。
2. 修改或檢查 `DeployDemo.Api.csproj、dotnet restore、dotnet build`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 3 的重點是：把「本機 restore/build 驗證」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 4 - 加入 Dockerfile build stage

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「加入 Dockerfile build stage」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「用 SDK image restore/publish 專案」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「加入 Dockerfile build stage」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：用 SDK image restore/publish 專案。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`Dockerfile build stage`。
- 資料與部署流向是：Docker build context -> COPY csproj -> restore -> COPY source -> publish。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「加入 Dockerfile build stage」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`Dockerfile build stage`。
- 資料怎麼流：Docker build context -> COPY csproj -> restore -> COPY source -> publish。
- 流程路線圖：需求 -> 找責任層 -> 修改 `Dockerfile build stage` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
docker build -t deploy-demo-api:day4 .
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「加入 Dockerfile build stage」。

具體流程：
1. 打開或建立 `Dockerfile build stage`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「加入 Dockerfile build stage」，不要跨層修太多東西。
2. 修改或檢查 `Dockerfile build stage`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 4 的重點是：把「加入 Dockerfile build stage」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 5 - 加入 Dockerfile runtime stage

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「加入 Dockerfile runtime stage」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「用 aspnet runtime image 跑已 publish 的 dll」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「加入 Dockerfile runtime stage」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：用 aspnet runtime image 跑已 publish 的 dll。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`Dockerfile final stage`。
- 資料與部署流向是：publish output -> runtime image /app -> dotnet DeployDemo.Api.dll。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「加入 Dockerfile runtime stage」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`Dockerfile final stage`。
- 資料怎麼流：publish output -> runtime image /app -> dotnet DeployDemo.Api.dll。
- 流程路線圖：需求 -> 找責任層 -> 修改 `Dockerfile final stage` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
docker run --rm deploy-demo-api:day5
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「加入 Dockerfile runtime stage」。

具體流程：
1. 打開或建立 `Dockerfile final stage`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「加入 Dockerfile runtime stage」，不要跨層修太多東西。
2. 修改或檢查 `Dockerfile final stage`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 5 的重點是：把「加入 Dockerfile runtime stage」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 6 - 理解 .NET container port 8080

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「理解 .NET container port 8080」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「讓 host port 與 container port 對齊，不再誤用舊範例 port 80」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「理解 .NET container port 8080」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：讓 host port 與 container port 對齊，不再誤用舊範例 port 80。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`Dockerfile EXPOSE、docker run -p、ASPNETCORE_HTTP_PORTS`。
- 資料與部署流向是：browser/curl -> host:8080 -> container:8080 -> Kestrel。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「理解 .NET container port 8080」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`Dockerfile EXPOSE、docker run -p、ASPNETCORE_HTTP_PORTS`。
- 資料怎麼流：browser/curl -> host:8080 -> container:8080 -> Kestrel。
- 流程路線圖：需求 -> 找責任層 -> 修改 `Dockerfile EXPOSE` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
docker run --rm -p 8080:8080 -e ASPNETCORE_HTTP_PORTS=8080 deploy-demo-api:day6
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「理解 .NET container port 8080」。

具體流程：
1. 打開或建立 `Dockerfile EXPOSE`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「理解 .NET container port 8080」，不要跨層修太多東西。
2. 修改或檢查 `Dockerfile EXPOSE、docker run -p、ASPNETCORE_HTTP_PORTS`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 6 的重點是：把「理解 .NET container port 8080」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 7 - 本機 docker build

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「本機 docker build」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「把 C# 專案變成 image」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「本機 docker build」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：把 C# 專案變成 image。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`Dockerfile、docker build`。
- 資料與部署流向是：source code -> docker build -> image layer -> local image store。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「本機 docker build」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`Dockerfile、docker build`。
- 資料怎麼流：source code -> docker build -> image layer -> local image store。
- 流程路線圖：需求 -> 找責任層 -> 修改 `Dockerfile` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
docker build -t deploy-demo-api:local .
docker images deploy-demo-api
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「本機 docker build」。

具體流程：
1. 打開或建立 `Dockerfile`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「本機 docker build」，不要跨層修太多東西。
2. 修改或檢查 `Dockerfile、docker build`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 7 的重點是：把「本機 docker build」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 8 - 本機 docker run

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「本機 docker run」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「從 image 建立 container 並驗證 /health」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「本機 docker run」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：從 image 建立 container 並驗證 /health。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`docker run、curl`。
- 資料與部署流向是：image -> container -> Kestrel -> /health JSON。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「本機 docker run」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`docker run、curl`。
- 資料怎麼流：image -> container -> Kestrel -> /health JSON。
- 流程路線圖：需求 -> 找責任層 -> 修改 `docker run` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
docker run --rm -p 8080:8080 deploy-demo-api:local
curl http://localhost:8080/health
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「本機 docker run」。

具體流程：
1. 打開或建立 `docker run`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「本機 docker run」，不要跨層修太多東西。
2. 修改或檢查 `docker run、curl`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 8 的重點是：把「本機 docker run」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 9 - 加入 docker compose

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「加入 docker compose」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「用 compose 保存本機啟動設定，減少手打指令」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「加入 docker compose」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：用 compose 保存本機啟動設定，減少手打指令。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`docker-compose.yml`。
- 資料與部署流向是：compose service -> build image -> port mapping -> API response。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「加入 docker compose」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`docker-compose.yml`。
- 資料怎麼流：compose service -> build image -> port mapping -> API response。
- 流程路線圖：需求 -> 找責任層 -> 修改 `docker-compose.yml` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
docker compose up --build
curl http://localhost:8080/orders/1
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「加入 docker compose」。

具體流程：
1. 打開或建立 `docker-compose.yml`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「加入 docker compose」，不要跨層修太多東西。
2. 修改或檢查 `docker-compose.yml`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 9 的重點是：把「加入 docker compose」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 10 - 規劃 image 命名

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「規劃 image 命名」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「把 image 命名成 GHCR 可接受格式」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「規劃 image 命名」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：把 image 命名成 GHCR 可接受格式。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`ghcr.io/<owner>/deploy-demo-api:<tag>`。
- 資料與部署流向是：repo owner -> image name -> tag -> registry package。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「規劃 image 命名」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`ghcr.io/<owner>/deploy-demo-api:<tag>`。
- 資料怎麼流：repo owner -> image name -> tag -> registry package。
- 流程路線圖：需求 -> 找責任層 -> 修改 `ghcr.io/<owner>/deploy-demo-api:<tag>` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
docker tag deploy-demo-api:local ghcr.io/YOUR_GITHUB_ACCOUNT/deploy-demo-api:local
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「規劃 image 命名」。

具體流程：
1. 打開或建立 `ghcr.io/<owner>/deploy-demo-api:<tag>`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「規劃 image 命名」，不要跨層修太多東西。
2. 修改或檢查 `ghcr.io/<owner>/deploy-demo-api:<tag>`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 10 的重點是：把「規劃 image 命名」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 11 - 手動登入 GHCR 觀念

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「手動登入 GHCR 觀念」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「理解本機 push 需要 PAT，Actions push 可用 GITHUB_TOKEN」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「手動登入 GHCR 觀念」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：理解本機 push 需要 PAT，Actions push 可用 GITHUB_TOKEN。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`docker login ghcr.io`。
- 資料與部署流向是：token -> docker login -> registry session -> push permission。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「手動登入 GHCR 觀念」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`docker login ghcr.io`。
- 資料怎麼流：token -> docker login -> registry session -> push permission。
- 流程路線圖：需求 -> 找責任層 -> 修改 `docker login ghcr.io` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
echo YOUR_PAT | docker login ghcr.io -u YOUR_GITHUB_ACCOUNT --password-stdin
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「手動登入 GHCR 觀念」。

具體流程：
1. 打開或建立 `docker login ghcr.io`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「手動登入 GHCR 觀念」，不要跨層修太多東西。
2. 修改或檢查 `docker login ghcr.io`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 11 的重點是：把「手動登入 GHCR 觀念」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 12 - 手動 tag / push 觀念

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「手動 tag / push 觀念」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「知道 Actions 自動化前，本機推 image 是怎麼運作」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「手動 tag / push 觀念」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：知道 Actions 自動化前，本機推 image 是怎麼運作。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`docker tag、docker push`。
- 資料與部署流向是：local image -> ghcr.io tag -> registry upload -> package。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「手動 tag / push 觀念」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`docker tag、docker push`。
- 資料怎麼流：local image -> ghcr.io tag -> registry upload -> package。
- 流程路線圖：需求 -> 找責任層 -> 修改 `docker tag` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
docker push ghcr.io/YOUR_GITHUB_ACCOUNT/deploy-demo-api:local
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「手動 tag / push 觀念」。

具體流程：
1. 打開或建立 `docker tag`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「手動 tag / push 觀念」，不要跨層修太多東西。
2. 修改或檢查 `docker tag、docker push`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 12 的重點是：把「手動 tag / push 觀念」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 13 - 建立 GitHub Actions workflow

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「建立 GitHub Actions workflow」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「用 YAML 定義 main push 觸發 build/push」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「建立 GitHub Actions workflow」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：用 YAML 定義 main push 觸發 build/push。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`.github/workflows/docker-publish.yml`。
- 資料與部署流向是：git push -> workflow trigger -> job -> steps。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「建立 GitHub Actions workflow」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`.github/workflows/docker-publish.yml`。
- 資料怎麼流：git push -> workflow trigger -> job -> steps。
- 流程路線圖：需求 -> 找責任層 -> 修改 `.github/workflows/docker-publish.yml` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
mkdir -p .github/workflows
# add .github/workflows/docker-publish.yml
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「建立 GitHub Actions workflow」。

具體流程：
1. 打開或建立 `.github/workflows/docker-publish.yml`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「建立 GitHub Actions workflow」，不要跨層修太多東西。
2. 修改或檢查 `.github/workflows/docker-publish.yml`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 13 的重點是：把「建立 GitHub Actions workflow」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 14 - 設定 workflow 權限

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「設定 workflow 權限」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「給 GITHUB_TOKEN packages: write，讓 workflow 可以推 GHCR」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「設定 workflow 權限」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：給 GITHUB_TOKEN packages: write，讓 workflow 可以推 GHCR。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`permissions: contents: read, packages: write`。
- 資料與部署流向是：workflow token -> package write permission -> ghcr.io push。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「設定 workflow 權限」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`permissions: contents: read, packages: write`。
- 資料怎麼流：workflow token -> package write permission -> ghcr.io push。
- 流程路線圖：需求 -> 找責任層 -> 修改 `permissions: contents: read, packages: write` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
permissions:
  contents: read
  packages: write
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「設定 workflow 權限」。

具體流程：
1. 打開或建立 `permissions: contents: read, packages: write`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「設定 workflow 權限」，不要跨層修太多東西。
2. 修改或檢查 `permissions: contents: read, packages: write`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 14 的重點是：把「設定 workflow 權限」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 15 - checkout 與 buildx

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「checkout 與 buildx」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「讓 runner 取得 repo 並啟用 Docker Buildx」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「checkout 與 buildx」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：讓 runner 取得 repo 並啟用 Docker Buildx。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`actions/checkout、docker/setup-buildx-action`。
- 資料與部署流向是：runner -> checkout code -> buildx builder -> Docker build。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「checkout 與 buildx」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`actions/checkout、docker/setup-buildx-action`。
- 資料怎麼流：runner -> checkout code -> buildx builder -> Docker build。
- 流程路線圖：需求 -> 找責任層 -> 修改 `actions/checkout` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
- uses: actions/checkout@v4
- uses: docker/setup-buildx-action@v3
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「checkout 與 buildx」。

具體流程：
1. 打開或建立 `actions/checkout`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「checkout 與 buildx」，不要跨層修太多東西。
2. 修改或檢查 `actions/checkout、docker/setup-buildx-action`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 15 的重點是：把「checkout 與 buildx」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 16 - 登入 GHCR action

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「登入 GHCR action」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「用 GitHub Actor 與 GITHUB_TOKEN 登入 ghcr.io」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「登入 GHCR action」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：用 GitHub Actor 與 GITHUB_TOKEN 登入 ghcr.io。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`docker/login-action@v3`。
- 資料與部署流向是：github.actor + GITHUB_TOKEN -> ghcr.io auth。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「登入 GHCR action」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`docker/login-action@v3`。
- 資料怎麼流：github.actor + GITHUB_TOKEN -> ghcr.io auth。
- 流程路線圖：需求 -> 找責任層 -> 修改 `docker/login-action@v3` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
- uses: docker/login-action@v3
  with:
    registry: ghcr.io
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「登入 GHCR action」。

具體流程：
1. 打開或建立 `docker/login-action@v3`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「登入 GHCR action」，不要跨層修太多東西。
2. 修改或檢查 `docker/login-action@v3`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 16 的重點是：把「登入 GHCR action」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 17 - build-push-action

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「build-push-action」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「用 Docker 官方 action build 並 push image」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「build-push-action」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：用 Docker 官方 action build 並 push image。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`docker/build-push-action@v6`。
- 資料與部署流向是：Dockerfile -> buildx -> image -> GHCR。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「build-push-action」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`docker/build-push-action@v6`。
- 資料怎麼流：Dockerfile -> buildx -> image -> GHCR。
- 流程路線圖：需求 -> 找責任層 -> 修改 `docker/build-push-action@v6` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
- uses: docker/build-push-action@v6
  with:
    push: true
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「build-push-action」。

具體流程：
1. 打開或建立 `docker/build-push-action@v6`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「build-push-action」，不要跨層修太多東西。
2. 修改或檢查 `docker/build-push-action@v6`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 17 的重點是：把「build-push-action」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 18 - latest 與 sha tag

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「latest 與 sha tag」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「同時提供易讀 tag 與可追蹤 commit tag」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「latest 與 sha tag」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：同時提供易讀 tag 與可追蹤 commit tag。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`tags: latest、${{ github.sha }}`。
- 資料與部署流向是：commit sha -> immutable tag -> deploy rollback reference。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「latest 與 sha tag」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`tags: latest、${{ github.sha }}`。
- 資料怎麼流：commit sha -> immutable tag -> deploy rollback reference。
- 流程路線圖：需求 -> 找責任層 -> 修改 `tags: latest` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
tags: |
  ${{ env.IMAGE_NAME }}:latest
  ${{ env.IMAGE_NAME }}:${{ github.sha }}
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「latest 與 sha tag」。

具體流程：
1. 打開或建立 `tags: latest`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「latest 與 sha tag」，不要跨層修太多東西。
2. 修改或檢查 `tags: latest、${{ github.sha }}`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 18 的重點是：把「latest 與 sha tag」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 19 - OCI source label

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「OCI source label」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「把 package 關聯回 GitHub repo」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「OCI source label」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：把 package 關聯回 GitHub repo。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`org.opencontainers.image.source`。
- 資料與部署流向是：image metadata -> package page -> source repo。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「OCI source label」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`org.opencontainers.image.source`。
- 資料怎麼流：image metadata -> package page -> source repo。
- 流程路線圖：需求 -> 找責任層 -> 修改 `org.opencontainers.image.source` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
labels: |
  org.opencontainers.image.source=${{ github.server_url }}/${{ github.repository }}
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「OCI source label」。

具體流程：
1. 打開或建立 `org.opencontainers.image.source`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「OCI source label」，不要跨層修太多東西。
2. 修改或檢查 `org.opencontainers.image.source`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 19 的重點是：把「OCI source label」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 20 - Actions 失敗排查

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「Actions 失敗排查」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「從 workflow log 判斷是 restore、build、login 還是 push 失敗」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「Actions 失敗排查」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：從 workflow log 判斷是 restore、build、login 還是 push 失敗。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`Actions log、job step`。
- 資料與部署流向是：failed step -> error output -> config fix。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「Actions 失敗排查」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`Actions log、job step`。
- 資料怎麼流：failed step -> error output -> config fix。
- 流程路線圖：需求 -> 找責任層 -> 修改 `Actions log` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
# Actions 頁面中先看第一個紅色 step 的 log
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「Actions 失敗排查」。

具體流程：
1. 打開或建立 `Actions log`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「Actions 失敗排查」，不要跨層修太多東西。
2. 修改或檢查 `Actions log、job step`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 20 的重點是：把「Actions 失敗排查」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 21 - Package visibility 與權限

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「Package visibility 與權限」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「理解 GHCR package 預設可能是 private，要調整 access」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「Package visibility 與權限」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：理解 GHCR package 預設可能是 private，要調整 access。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`GitHub Packages settings`。
- 資料與部署流向是：package -> visibility/access -> pull permission。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「Package visibility 與權限」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`GitHub Packages settings`。
- 資料怎麼流：package -> visibility/access -> pull permission。
- 流程路線圖：需求 -> 找責任層 -> 修改 `GitHub Packages settings` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
# GitHub Packages -> package -> Package settings -> Manage Actions access
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「Package visibility 與權限」。

具體流程：
1. 打開或建立 `GitHub Packages settings`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「Package visibility 與權限」，不要跨層修太多東西。
2. 修改或檢查 `GitHub Packages settings`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 21 的重點是：把「Package visibility 與權限」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 22 - 遠端主機 pull image

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「遠端主機 pull image」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「部署主機只需要 pull 已發布 image，不一定要有 source code」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「遠端主機 pull image」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：部署主機只需要 pull 已發布 image，不一定要有 source code。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`docker pull ghcr.io/<owner>/deploy-demo-api:<sha>`。
- 資料與部署流向是：GHCR -> docker host -> local image。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「遠端主機 pull image」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`docker pull ghcr.io/<owner>/deploy-demo-api:<sha>`。
- 資料怎麼流：GHCR -> docker host -> local image。
- 流程路線圖：需求 -> 找責任層 -> 修改 `docker pull ghcr.io/<owner>/deploy-demo-api:<sha>` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
docker pull ghcr.io/YOUR_GITHUB_ACCOUNT/deploy-demo-api:<commit-sha>
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「遠端主機 pull image」。

具體流程：
1. 打開或建立 `docker pull ghcr.io/<owner>/deploy-demo-api:<sha>`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「遠端主機 pull image」，不要跨層修太多東西。
2. 修改或檢查 `docker pull ghcr.io/<owner>/deploy-demo-api:<sha>`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 22 的重點是：把「遠端主機 pull image」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 23 - 遠端主機 run image

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「遠端主機 run image」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「用 docker run 驗證 image 在主機可啟動」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「遠端主機 run image」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：用 docker run 驗證 image 在主機可啟動。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`docker run -p 8080:8080`。
- 資料與部署流向是：container -> port mapping -> curl /health。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「遠端主機 run image」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`docker run -p 8080:8080`。
- 資料怎麼流：container -> port mapping -> curl /health。
- 流程路線圖：需求 -> 找責任層 -> 修改 `docker run -p 8080:8080` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
docker run -d --name deploy-demo-api -p 8080:8080 ghcr.io/YOUR_GITHUB_ACCOUNT/deploy-demo-api:<commit-sha>
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「遠端主機 run image」。

具體流程：
1. 打開或建立 `docker run -p 8080:8080`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「遠端主機 run image」，不要跨層修太多東西。
2. 修改或檢查 `docker run -p 8080:8080`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 23 的重點是：把「遠端主機 run image」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 24 - 遠端 compose 使用 GHCR image

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「遠端 compose 使用 GHCR image」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「把 build 改成 image，讓部署端拉取 GHCR 成品」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「遠端 compose 使用 GHCR image」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：把 build 改成 image，讓部署端拉取 GHCR 成品。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`docker-compose.prod.yml`。
- 資料與部署流向是：compose pull -> compose up -> API container。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「遠端 compose 使用 GHCR image」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`docker-compose.prod.yml`。
- 資料怎麼流：compose pull -> compose up -> API container。
- 流程路線圖：需求 -> 找責任層 -> 修改 `docker-compose.prod.yml` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「遠端 compose 使用 GHCR image」。

具體流程：
1. 打開或建立 `docker-compose.prod.yml`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「遠端 compose 使用 GHCR image」，不要跨層修太多東西。
2. 修改或檢查 `docker-compose.prod.yml`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 24 的重點是：把「遠端 compose 使用 GHCR image」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 25 - 環境變數與 secrets 邊界

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「環境變數與 secrets 邊界」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「分清楚 GitHub Secrets、container env、appsettings」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「環境變數與 secrets 邊界」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：分清楚 GitHub Secrets、container env、appsettings。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`GitHub Secrets、.env、ASPNETCORE_HTTP_PORTS`。
- 資料與部署流向是：secret source -> workflow/deploy env -> app config。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「環境變數與 secrets 邊界」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`GitHub Secrets、.env、ASPNETCORE_HTTP_PORTS`。
- 資料怎麼流：secret source -> workflow/deploy env -> app config。
- 流程路線圖：需求 -> 找責任層 -> 修改 `GitHub Secrets` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
ASPNETCORE_HTTP_PORTS=8080
ConnectionStrings__Default=placeholder-only
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「環境變數與 secrets 邊界」。

具體流程：
1. 打開或建立 `GitHub Secrets`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「環境變數與 secrets 邊界」，不要跨層修太多東西。
2. 修改或檢查 `GitHub Secrets、.env、ASPNETCORE_HTTP_PORTS`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 25 的重點是：把「環境變數與 secrets 邊界」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 26 - 加上簡單 smoke test

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「加上簡單 smoke test」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「在 workflow build 後至少檢查 image 可建立或 API 可回應」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「加上簡單 smoke test」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：在 workflow build 後至少檢查 image 可建立或 API 可回應。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`docker run、curl /health`。
- 資料與部署流向是：image -> test container -> HTTP check -> fail fast。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「加上簡單 smoke test」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`docker run、curl /health`。
- 資料怎麼流：image -> test container -> HTTP check -> fail fast。
- 流程路線圖：需求 -> 找責任層 -> 修改 `docker run` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
docker run -d --name smoke -p 8080:8080 $IMAGE
curl --fail http://localhost:8080/health
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「加上簡單 smoke test」。

具體流程：
1. 打開或建立 `docker run`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「加上簡單 smoke test」，不要跨層修太多東西。
2. 修改或檢查 `docker run、curl /health`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 26 的重點是：把「加上簡單 smoke test」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 27 - 版本回滾策略

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「版本回滾策略」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「用 sha tag 回到上一版 image」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「版本回滾策略」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：用 sha tag 回到上一版 image。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`ghcr.io/<owner>/deploy-demo-api:<old-sha>`。
- 資料與部署流向是：incident -> choose known tag -> compose update -> restart。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「版本回滾策略」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`ghcr.io/<owner>/deploy-demo-api:<old-sha>`。
- 資料怎麼流：incident -> choose known tag -> compose update -> restart。
- 流程路線圖：需求 -> 找責任層 -> 修改 `ghcr.io/<owner>/deploy-demo-api:<old-sha>` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
docker compose pull api
docker compose up -d api
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「版本回滾策略」。

具體流程：
1. 打開或建立 `ghcr.io/<owner>/deploy-demo-api:<old-sha>`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「版本回滾策略」，不要跨層修太多東西。
2. 修改或檢查 `ghcr.io/<owner>/deploy-demo-api:<old-sha>`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 27 的重點是：把「版本回滾策略」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 28 - 不要把小專案推到使用者 GitHub

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「不要把小專案推到使用者 GitHub」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「本任務只把專案作為筆記中的可複製範例」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「不要把小專案推到使用者 GitHub」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：本任務只把專案作為筆記中的可複製範例。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`Markdown code blocks`。
- 資料與部署流向是：temp demo -> verified snippets -> note。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「不要把小專案推到使用者 GitHub」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`Markdown code blocks`。
- 資料怎麼流：temp demo -> verified snippets -> note。
- 流程路線圖：需求 -> 找責任層 -> 修改 `Markdown code blocks` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
# 本任務不把 src/.github 小專案提交，只提交 Markdown 筆記
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「不要把小專案推到使用者 GitHub」。

具體流程：
1. 打開或建立 `Markdown code blocks`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「不要把小專案推到使用者 GitHub」，不要跨層修太多東西。
2. 修改或檢查 `Markdown code blocks`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 28 的重點是：把「不要把小專案推到使用者 GitHub」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 29 - 整理交付 checklist

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「整理交付 checklist」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「把 GitHub Actions 上線前檢查寫成清單」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「整理交付 checklist」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：把 GitHub Actions 上線前檢查寫成清單。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`README、Actions tab、Packages tab`。
- 資料與部署流向是：repo files -> push -> workflow -> package -> pull/run。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「整理交付 checklist」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`README、Actions tab、Packages tab`。
- 資料怎麼流：repo files -> push -> workflow -> package -> pull/run。
- 流程路線圖：需求 -> 找責任層 -> 修改 `README` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
git push origin main
# Actions -> Packages -> docker pull -> docker run -> curl
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「整理交付 checklist」。

具體流程：
1. 打開或建立 `README`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「整理交付 checklist」，不要跨層修太多東西。
2. 修改或檢查 `README、Actions tab、Packages tab`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 29 的重點是：把「整理交付 checklist」變成 C# Docker 部署流程中可驗證、可交接的一步。

## Day 30 - 完整端到端復盤

### 這篇文章主要在講什麼

這一天把原 Docker 部署主線轉成 C# 版本的「完整端到端復盤」。重點不是多背一個指令，而是讓 DeployDemo.Api 往「把原 Docker 部署主線轉成 C# + GHCR 可實作流程」前進。

### 為什麼需要這個概念

如果沒有先理解這一天的責任邊界，部署失敗時會混在一起：可能是 C# build 失敗、Dockerfile 寫錯、GitHub Actions 權限不足、GHCR package 權限不對，或遠端主機 port 沒開。今天要把問題切到「完整端到端復盤」這一層。

### 學完這篇你應該會做到什麼

你應該能完成並驗證：把原 Docker 部署主線轉成 C# + GHCR 可實作流程。完成不是「檔案有寫」，而是能用命令、log、HTTP response 或 GitHub Actions 畫面證明它真的能跑。

### 核心重點

- 會動到的核心範圍是：`Program.cs、Dockerfile、workflow、compose`。
- 資料與部署流向是：C# API -> Docker image -> GHCR -> remote host -> health check。
- 每一步都要留下可重跑的驗證方式，避免只靠印象判斷成功。

### 真實工作流程例子

- 工作任務：主管希望把 C# API 的 Docker 部署流程補到 GitHub Actions，今天負責「完整端到端復盤」這一段。
- 你先判斷：先確認這次是在改 C# 專案、Dockerfile、compose、workflow YAML、GHCR 權限，還是遠端主機啟動方式，不要直接貼整包範例。
- 會動到：`Program.cs、Dockerfile、workflow、compose`。
- 資料怎麼流：C# API -> Docker image -> GHCR -> remote host -> health check。
- 流程路線圖：需求 -> 找責任層 -> 修改 `Program.cs` -> 執行驗證 -> 記錄失敗排查。
- 工作中會寫 / 檢查的片段：

```bash
curl --fail http://localhost:8080/health
docker logs deploy-demo-api --tail 50
```

- 交付前驗證：至少確認 happy path 成功；再檢查一個失敗情境，例如 build 失敗、port 不通、token 權限不足、image tag 不存在。
- 常見卡點：看到錯誤先定位是哪一層，不要同時改 C#、Dockerfile、workflow、遠端主機設定。

### 主線專案銜接

今天銜接主線專案的「完整端到端復盤」。

具體流程：
1. 打開或建立 `Program.cs`，確認今天的改動入口。
2. 套用本日範例，但保留 `YOUR_GITHUB_ACCOUNT`、secret、domain 這類 placeholder。
3. 執行本日驗證命令，記錄成功輸出與失敗排查方向。

具體檢查：
- 檔案位置正確，沒有把 workflow 放錯資料夾。
- 命令在專案根目錄或正確子目錄執行。
- 驗證方式能證明今天的交付物，而不是只證明檔案存在。

### 當天做完後檢查

- 可以說明今天處理的是 C#、Docker、GitHub Actions、GHCR 或遠端部署哪一層。
- 可以指出至少一個成功輸出與一個失敗排查點。
- 沒有把真實 token、密碼或私有主機資訊寫進 repo。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `Program.cs`、`Dockerfile`、workflow env、GHCR image tag | 定義 API、建置方式、image 名稱與部署設定 |
| 流程或邏輯 | `dotnet build`、`docker build`、GitHub Actions job | 把 source code 轉成可發布 image |
| 使用端或呈現 | `curl /health`、Actions log、Packages 頁面、遠端 `docker ps` | 驗證服務與 image 是否真的可用 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天要處理「完整端到端復盤」，不要跨層修太多東西。
2. 修改或檢查 `Program.cs、Dockerfile、workflow、compose`。
3. 執行本日命令或在 GitHub Actions 畫面確認對應 step。
4. 用另一種方式驗證，例如 build 後看 output、push 後看 package、run 後打 `/health`。
5. 把錯誤訊息回填到排查筆記，不要只留下成功指令。

注意事項：
- GitHub Actions 裡可以用 `GITHUB_TOKEN` 推同 repo 關聯的 GHCR package，但 package 權限仍要檢查。
- ASP.NET Core 官方 container image 常見預設 port 是 `8080`，host port mapping 要看清楚 `host:container`。
- demo code 可以練習，但正式環境還需要 logging、health check、secret management、rollback 與監控。

### 如果結果和預期不同

- C# build 失敗：先看 compiler error，不要急著改 Dockerfile。
- Docker build 失敗：看是哪一個 Dockerfile layer 失敗，尤其是 `COPY` 路徑與 `dotnet restore`。
- Actions push 失敗：先看 `permissions`、login step、package access，不要直接換成個人 token。

### 負面例子 / 錯誤用法

錯誤做法：把原本 Docker 教學的指令直接套到 C# 專案，沒有檢查 `.csproj` 路徑、container port、workflow 權限與 GHCR image 名稱。

問題：
- build context 可能找不到 `.csproj`。
- container 其實跑在 8080，但你對外打 80。
- workflow 有 build 成功，卻因 `packages: write` 缺少而 push 失敗。

修正方向：每一層都留一個獨立驗證點，先縮小失敗範圍再改設定。

### 小練習

把今天的主題改一個小地方：例如改 endpoint 回傳欄位、換 image tag、或故意拿掉 `packages: write`，觀察錯誤會出現在 C#、Docker 還是 GitHub Actions 哪一層。

### Junior 常見誤解

常見誤解是以為「workflow 綠了」就等於部署完成。實際上 workflow 綠了只代表 image 發布成功，遠端主機是否 pull 到正確 tag、container 是否能對外回應，還要另外驗證。

### 一句話總結

Day 30 的重點是：把「完整端到端復盤」變成 C# Docker 部署流程中可驗證、可交接的一步。


