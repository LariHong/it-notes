# 用 Docker 了嗎？30 天 Docker 基本教學：C# / ASP.NET Core 2026 更新整理

## 來源

- 使用者提供的系列頁：https://ithelp.ithome.com.tw/m/users/20140124/ironman/5149
- 原系列 RSS：https://ithelp.ithome.com.tw/rss/series/5149
- 原系列名稱：用 Docker 了嗎？30 天的 Docker 基本教學
- 原作者：Robert Chang
- 原系列時間：2022-09-16 到 2022-10-15
- 2026 更新參考：
  - Docker 官方概念文件：https://docs.docker.com/get-started/docker-overview/
  - .NET 與 Docker 官方文件：https://learn.microsoft.com/en-us/dotnet/core/docker/introduction
  - .NET 支援週期：https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core

> 來源提醒：使用者提供的連結是 Docker 系列，不是 C# 系列。這份筆記會保留原系列的 Docker 學習順序，但以 C# / ASP.NET Core 開發者的角度重新整理，範例也以 C# 為主。

## 這份筆記怎麼讀

如果你從沒用過 Docker，可以先把 Docker 想成「把你的 C# 程式、執行環境、設定方式包成一個可搬運的盒子」。沒有 Docker 時，常見狀況是「我電腦能跑，你電腦不能跑」、「測試機少裝一個 runtime」、「資料庫版本不同」。Docker 要解決的就是這種環境不一致。

這份筆記建議用下面順序學：

1. 先理解 container、image、registry、volume、network 這些名詞。
2. 用 ASP.NET Core minimal API 做最小 C# 專案。
3. 把 C# 專案包成 Docker image。
4. 用 Docker Compose 串 PostgreSQL 或 SQL Server。
5. 學會 log、環境變數、port、volume、.dockerignore。
6. 最後再碰 registry、CI/CD、Kubernetes 或雲端部署。

## 2026 年讀原系列要更新的地方

- .NET 版本：截至 2026-05-03，.NET 10 是 LTS，.NET 9 是 STS，.NET 8 仍在支援週期內。新專案可優先選 .NET 10，工作專案常見仍會遇到 .NET 8。
- ASP.NET Core container port：.NET 8 之後官方 ASP.NET Core container image 預設聽 `8080`，不是舊文章或舊範例常見的 `80`。
- 官方 image：.NET 官方 image 主要在 Microsoft Artifact Registry，例如 `mcr.microsoft.com/dotnet/aspnet:10.0` 與 `mcr.microsoft.com/dotnet/sdk:10.0`。
- 安全性：.NET 8 之後官方 image 內建 `app` 非 root 使用者，正式環境應避免讓 app 以 root 跑。
- 建置方式：除了 Dockerfile，.NET SDK 也支援 `dotnet publish -t:PublishContainer` 產生 container image，但初學者仍建議先學 Dockerfile，因為它能建立 Docker 的基本心智模型。
- Compose：`docker compose` 已是 v2 指令，現在通常不再寫成舊式的 `docker-compose`。

## C# 開發者的最小練習專案

後面每個 Day 都可以用這個 minimal API 當練習基礎。

```powershell
dotnet new webapi -n DockerCSharpDemo
cd DockerCSharpDemo
dotnet run
```

把 `Program.cs` 改成更小的版本：

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// 範例用途：提供一個最小 API，讓 Docker 練習可以用 HTTP 驗證容器是否真的在跑。
// 輸入：GET /health，沒有 request body。
// 回傳結果：HTTP 200 與目前時間，方便確認 container 內的程式有回應。
app.MapGet("/health", () => new
{
    Status = "ok",
    Runtime = System.Runtime.InteropServices.RuntimeInformation.FrameworkDescription,
    Now = DateTimeOffset.UtcNow
});

app.Run();
```

最小 Dockerfile：

```dockerfile
# Build stage：使用 SDK image 編譯 C# 專案。
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY *.csproj ./
RUN dotnet restore
COPY . ./
RUN dotnet publish -c Release -o /app/publish

# Runtime stage：只帶 ASP.NET Core runtime，不把 SDK 放進正式 image。
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS final
WORKDIR /app
COPY --from=build /app/publish .
USER app
EXPOSE 8080
ENTRYPOINT ["dotnet", "DockerCSharpDemo.dll"]
```

執行：

```powershell
docker build -t docker-csharp-demo:dev .
docker run --rm -p 8080:8080 docker-csharp-demo:dev
curl http://localhost:8080/health
```

## 整體地圖

| 階段 | Day | 原系列主題 | C# 開發者要學會什麼 |
| --- | --- | --- | --- |
| 建立直覺 | 1-5 | Docker 來源、安裝、架構、基本概念 | 知道 Docker 解決 C# 專案環境不一致問題 |
| 操作容器 | 6-13 | container 啟動、指令、內部狀態 | 會跑 ASP.NET Core container、看 log、進容器檢查 |
| 網路 | 14-17 | network、port、DNS | 會讓 API 與資料庫用 service name 溝通 |
| Image | 18-23 | image、tag、registry | 會替 C# API 建 image、打 tag、推到 registry |
| Dockerfile | 24-28 | Dockerfile、build cache、multi-stage、ignore | 會寫安全且快取友善的 .NET Dockerfile |
| Volume | 29-30 | volume 與資料持久化 | 會用 volume 保存資料庫資料，不把狀態塞進 container |

## 來源清單

| Day | 標題 | 連結 |
| --- | --- | --- |
| 1 | Docker 怎麼來的？ | https://ithelp.ithome.com.tw/articles/10291348 |
| 2 | Docker 之旅的行前須知 | https://ithelp.ithome.com.tw/articles/10292468 |
| 3 | 安裝 Docker | https://ithelp.ithome.com.tw/articles/10292988 |
| 4 | Docker 的基礎架構 | https://ithelp.ithome.com.tw/articles/10293745 |
| 5 | 所以說 Docker 是什麼？ | https://ithelp.ithome.com.tw/articles/10294528 |
| 6 | 啟動 Docker Container | https://ithelp.ithome.com.tw/articles/10295219 |
| 7 | Docker Container 啟動時發生了什麼？ | https://ithelp.ithome.com.tw/articles/10295968 |
| 8 | Docker Container 的相關操作指令（上） | https://ithelp.ithome.com.tw/articles/10296644 |
| 9 | Docker Container 的相關操作指令（中） | https://ithelp.ithome.com.tw/articles/10297304 |
| 10 | Docker Container 的相關操作指令（下） | https://ithelp.ithome.com.tw/articles/10298001 |
| 11 | Docker Container vs 虛擬機 | https://ithelp.ithome.com.tw/articles/10298643 |
| 12 | 深入 Docker Container 內部（上） | https://ithelp.ithome.com.tw/articles/10299117 |
| 13 | 深入 Docker Container 內部（下） | https://ithelp.ithome.com.tw/articles/10300030 |
| 14 | Docker 的網路世界（上） | https://ithelp.ithome.com.tw/articles/10300587 |
| 15 | Docker 的網路世界（下） | https://ithelp.ithome.com.tw/articles/10301305 |
| 16 | Docker Network 指令 | https://ithelp.ithome.com.tw/articles/10301998 |
| 17 | Docker 中的 DNS | https://ithelp.ithome.com.tw/articles/10302490 |
| 18 | 什麼是映像檔（Docker Image）？ | https://ithelp.ithome.com.tw/articles/10303158 |
| 19 | Docker Image 的標籤 | https://ithelp.ithome.com.tw/articles/10303720 |
| 20 | Docker Image 快取的秘密 | https://ithelp.ithome.com.tw/articles/10303837 |
| 21 | Docker Image 的唯讀性 | https://ithelp.ithome.com.tw/articles/10304292 |
| 22 | 推送映像檔至 DockerHub | https://ithelp.ithome.com.tw/articles/10304380 |
| 23 | Docker Image 的完全名稱及 Docker Registry | https://ithelp.ithome.com.tw/articles/10304911 |
| 24 | Dockerfile 指令解析 | https://ithelp.ithome.com.tw/articles/10306303 |
| 25 | 建置 Docker Image | https://ithelp.ithome.com.tw/articles/10307063 |
| 26 | 建置 Docker Image 的快取機制 | https://ithelp.ithome.com.tw/articles/10307076 |
| 27 | 多階段建置 Docker Image | https://ithelp.ithome.com.tw/articles/10307346 |
| 28 | .dockerignore 該怎麼使用？ | https://ithelp.ithome.com.tw/articles/10308060 |
| 29 | 介紹 Docker Volume | https://ithelp.ithome.com.tw/articles/10308403 |
| 30 | 實際使用 Docker Volume | https://ithelp.ithome.com.tw/articles/10308405 |

---

## Day 1：Docker 怎麼來的？

### 這篇文章主要在講什麼

原文從 Docker 的歷史開始：以前部署程式常常要手動裝環境、設定伺服器、處理版本差異。Docker 讓應用程式可以被包成 image，再用 container 跑起來。

### 為什麼 C# 開發者需要它

C# 專案通常依賴 .NET runtime、環境變數、資料庫、憑證、port 設定。Docker 可以把「怎麼跑」寫進專案，不再只靠口頭文件。

### 直觀例子

沒有 Docker 像是每次搬家都重新買家具；Docker image 像是打包好的露營車，到哪裡都用同一套配置。

### C# 練習

```powershell
dotnet new webapi -n Day01DockerWhy
cd Day01DockerWhy
dotnet run
```

先不要急著 containerize。你要先感受：現在這個 API 能跑，是因為你本機剛好有 .NET SDK。

### 負面例子 / 錯誤用法

錯誤做法：只在 README 寫「請安裝最新版 .NET」。

問題：每個人的最新版可能不同，CI 和正式機也可能不同。

修正方向：在 `global.json`、Dockerfile、CI 中明確指定版本，例如 .NET 10 或 .NET 8。

### 一句話總結

Docker 的核心價值不是炫技，而是讓 C# 程式的執行環境可以被描述、重建與搬運。

## Day 2：Docker 之旅的行前須知

### 這篇文章主要在講什麼

原文提醒要準備 Docker Hub、GitHub、VS Code extension 與終端機基本概念。

### C# 開發者準備清單

1. 安裝 Docker Desktop 或 Docker Engine。
2. 安裝 .NET SDK 10 或專案指定版本。
3. 安裝 Git。
4. 準備 Docker Hub、GitHub Container Registry 或公司 registry 帳號。
5. 編輯器可用 Visual Studio、Rider、VS Code。

### C# 練習

```powershell
dotnet --info
docker version
docker compose version
git --version
```

### 負面例子 / 錯誤用法

錯誤做法：還沒確認 Docker daemon 有啟動就一直改 Dockerfile。

問題：錯誤不在程式，而是 Docker 服務沒跑。

修正方向：先跑 `docker version`，確認 Client 和 Server 都有資訊。

### 一句話總結

學 Docker 前先把工具鏈確認好，避免把環境問題誤判成 C# 程式問題。

## Day 3：安裝 Docker

### 這篇文章主要在講什麼

原文介紹 Windows、macOS、Linux 的 Docker 安裝方式。

### 2026 C# 建議

Windows 開發 C# 最常見是 Docker Desktop + WSL 2。ASP.NET Core Linux container 是主流選擇，除非你真的需要 Windows-only API，才考慮 Windows container。

### 完整流程

1. 安裝 Docker Desktop。
2. 啟用 WSL 2 backend。
3. 打開 Docker Desktop，等待狀態變成 running。
4. 執行 `docker run hello-world`。
5. 執行 C# minimal API container 測試 port。

```powershell
docker run --rm hello-world
docker run --rm -p 8080:8080 mcr.microsoft.com/dotnet/samples:aspnetapp
```

### 負面例子 / 錯誤用法

錯誤做法：看到 Windows 就選 Windows container。

問題：大多數 ASP.NET Core 專案在 Linux container 更輕、更常見，也更接近雲端部署環境。

修正方向：初學者先用 Linux container。

### 一句話總結

先讓 Docker 本身能跑，再開始處理 C# 專案 container 化。

## Day 4：Docker 的基礎架構

### 核心概念

| 名稱 | C# 開發者可以怎麼想 |
| --- | --- |
| Docker client | 你輸入 `docker` 指令的工具 |
| Docker daemon | 真正建立 image、啟動 container 的背景服務 |
| Image | C# app 加 runtime 的可重建模板 |
| Container | image 跑起來後的一個執行個體 |
| Registry | 放 image 的倉庫，例如 Docker Hub、GHCR、ACR |

### C# 練習

```powershell
docker pull mcr.microsoft.com/dotnet/aspnet:10.0
docker image ls
```

### 負面例子 / 錯誤用法

錯誤做法：把 container 當成一台小 VM，進去手動裝套件、手動修程式。

問題：container 刪掉後變更就不見，且無法重現。

修正方向：所有變更寫進 Dockerfile 或原始碼。

### 一句話總結

Docker 的世界裡，image 是模板，container 是執行中的實例。

## Day 5：所以說 Docker 是什麼？

### 核心概念

Docker 是開發、打包、分享與執行應用程式的平台。對 C# 來說，它讓 ASP.NET Core API 可以用一致方式在本機、CI、測試機與正式環境跑起來。

### C# 練習

建立 `Program.cs`：

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// 範例用途：用最小 endpoint 驗證 Docker container 是否能回應 HTTP。
// 回傳結果：字串內容會顯示在 curl 或瀏覽器。
app.MapGet("/", () => "Hello from ASP.NET Core in Docker");

app.Run();
```

### 不適合使用 Docker 的情境

- 你只是剛開始學 C# 變數、if、loop，還不需要部署環境。
- 你的專案完全依賴 Windows 桌面 UI，例如 WinForms 入門練習。
- 團隊尚未理解 image 與資料保存，直接把正式資料放進 container 內。

### 一句話總結

Docker 是把「程式如何被執行」變成專案一部分的工具。

## Day 6：啟動 Docker Container

### 核心指令

```powershell
docker run --rm -p 8080:8080 docker-csharp-demo:dev
docker ps
docker stop <container-id>
```

### C# 操作流程

1. 建立 C# API。
2. 建立 Dockerfile。
3. `docker build` 產生 image。
4. `docker run` 啟動 container。
5. 用 `curl http://localhost:8080/health` 驗證。

### 負面例子 / 錯誤用法

錯誤做法：`docker run docker-csharp-demo:dev` 後直接打開 `localhost:8080`。

問題：沒有 `-p 8080:8080` 就沒有把 container port 發布到 host。

修正方向：理解 `hostPort:containerPort`。

### 一句話總結

container 不是啟動就等於外部可連，要明確 publish port。

## Day 7：Container 啟動時發生了什麼？

### 核心概念

`docker run` 會檢查 image、建立 container、設定檔案系統、網路、環境變數，最後執行 `ENTRYPOINT` 或 `CMD`。

### C# Dockerfile 對應

```dockerfile
ENTRYPOINT ["dotnet", "DockerCSharpDemo.dll"]
```

這行代表 container 啟動後會執行 C# app。

### 負面例子 / 錯誤用法

錯誤做法：DLL 名稱跟專案輸出不一致。

問題：container 會立刻退出，log 顯示找不到檔案。

修正方向：用 `docker logs <container>` 看錯誤，再檢查 `.csproj` 名稱與 `ENTRYPOINT`。

### 一句話總結

container 啟動的最後一步，就是執行你在 image 裡指定的主程式。

## Day 8：Container 操作指令（上）

### 常用指令

```powershell
docker ps
docker ps -a
docker logs <container>
docker inspect <container>
```

### C# 練習

```csharp
app.MapGet("/config", (IConfiguration config) => new
{
    Environment = app.Environment.EnvironmentName,
    Message = config["DemoMessage"] ?? "not set"
});
```

```powershell
docker run --rm -p 8080:8080 -e DemoMessage="hello config" docker-csharp-demo:dev
```

### 負面例子 / 錯誤用法

錯誤做法：把連線字串直接寫在 `Program.cs`。

問題：image 推到 registry 後，秘密也跟著外流。

修正方向：用環境變數、secret manager、雲端 secret 服務。

### 一句話總結

`logs`、`inspect`、環境變數，是排查 C# container 的基本工具。

## Day 9：Container 操作指令（中）

### 常用指令

```powershell
docker exec -it <container> sh
docker cp <container>:/app/appsettings.json .
docker stats
```

### C# 練習

進入 container 後檢查檔案：

```powershell
docker exec -it <container> sh
ls -la /app
```

### 負面例子 / 錯誤用法

錯誤做法：進 container 改 `appsettings.json` 來修正式問題。

問題：下一次重建或重啟就消失，也沒有版本紀錄。

修正方向：用環境變數或掛載設定檔，並把正式設定交給部署平台管理。

### 一句話總結

可以進 container 檢查，但不應該把手動修改 container 當部署方式。

## Day 10：Container 操作指令（下）

### 常用指令

```powershell
docker rm <container>
docker rm -f <container>
docker container prune
docker image prune
```

### C# 開發者注意

清掉 container 不會清掉你建好的 image，也不會一定清掉 volume。這對資料庫練習很重要。

### 負面例子 / 錯誤用法

錯誤做法：看到很多 container 就亂跑 `docker system prune -a --volumes`。

問題：image cache、資料庫 volume 可能都被清掉。

修正方向：先用 `docker ps -a`、`docker volume ls` 確認再清。

### 一句話總結

清理 Docker 資源前，要知道自己正在刪 container、image 還是 volume。

## Day 11：Container vs 虛擬機

### 核心差異

VM 包一整套作業系統；container 共用 host kernel，只包應用程式需要的檔案與隔離環境。

### C# 實務判斷

- ASP.NET Core API：適合 container。
- 背景 worker：適合 container。
- WinForms / WPF 桌面程式：通常不適合 container 作為使用者端部署。
- 舊 .NET Framework + Windows-only 元件：可能需要 Windows container 或維持 VM。

### 負面例子 / 錯誤用法

錯誤做法：把 container 當成完整伺服器，放 SSH、排程、資料庫、API 全部塞同一個 container。

問題：難維護、難擴充、難監控。

修正方向：一個 container 優先負責一個主要 process。

### 一句話總結

container 比 VM 輕，但它不是拿來取代所有伺服器管理觀念的魔法。

## Day 12：深入 Container 內部（上）

### 核心概念

container 的檔案系統來自 image layer，加上一層可寫層。container 被刪除後，可寫層通常也消失。

### C# 練習

```csharp
app.MapPost("/temp-file", () =>
{
    var path = Path.Combine(Path.GetTempPath(), "demo.txt");
    File.WriteAllText(path, DateTimeOffset.UtcNow.ToString("O"));
    return Results.Ok(new { Path = path });
});
```

這個檔案只存在 container 生命週期中，不適合存正式資料。

### 負面例子 / 錯誤用法

錯誤做法：把使用者上傳檔案存到 container 的 `/app/uploads`。

問題：container 被重建後檔案可能消失。

修正方向：使用 object storage、volume 或外部檔案服務。

### 一句話總結

container 內的可寫層是短暫的，不要把它當長期儲存。

## Day 13：深入 Container 內部（下）

### 核心概念

container 有隔離的 process、network、filesystem，但不是絕對安全邊界。權限、user、image 來源仍然重要。

### C# 安全建議

```dockerfile
USER app
```

正式 image 不要用 root 執行 ASP.NET Core app。

### 負面例子 / 錯誤用法

錯誤做法：為了方便 debug，把 production image 裝滿 shell、curl、vim、root 權限。

問題：攻擊面增加，image 變大，也容易和正式環境安全要求衝突。

修正方向：debug image 和 production image 分開；正式 image 使用較小、非 root 的 runtime image。

### 一句話總結

container 有隔離，但安全仍要靠最小權限與乾淨 image。

## Day 14：Docker 的網路世界（上）

### 核心概念

container 有自己的網路命名空間。對外提供服務時，用 port mapping；container 彼此溝通時，通常放在同一個 Docker network。

### C# 練習

```powershell
docker network create demo-net
docker run --rm --network demo-net --name api -p 8080:8080 docker-csharp-demo:dev
```

### 負面例子 / 錯誤用法

錯誤做法：在 container 裡用 `localhost` 連另一個 container 的 PostgreSQL。

問題：`localhost` 是「自己這個 container」，不是別的 container。

修正方向：用 Compose service name，例如 `Host=postgres`。

### 一句話總結

在 container 裡，`localhost` 指的是自己，不是你的電腦，也不是其他服務。

## Day 15：Docker 的網路世界（下）

### C# + PostgreSQL Compose 範例

```yaml
services:
  api:
    build: .
    ports:
      - "8080:8080"
    environment:
      ConnectionStrings__Default: "Host=db;Port=5432;Database=demo;Username=postgres;Password=postgres"
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: demo
    volumes:
      - pg-data:/var/lib/postgresql/data

volumes:
  pg-data:
```

### 負面例子 / 錯誤用法

錯誤做法：API connection string 寫 `Host=localhost`。

問題：API container 會連自己，不會連到 PostgreSQL container。

修正方向：在 Compose 裡用 `Host=db`，因為 `db` 是 service name。

### 一句話總結

Compose 裡的 service name 就是 container 之間的 DNS 名稱。

## Day 16：Docker Network 指令

### 常用指令

```powershell
docker network ls
docker network inspect <network>
docker network create app-net
docker network rm app-net
```

### C# 操作流程

1. 建立 network。
2. 啟動資料庫 container。
3. 啟動 API container 並加入同一個 network。
4. 用資料庫 container name 作為 host。

### 負面例子 / 錯誤用法

錯誤做法：所有服務都掛到 default bridge，靠手動 IP 連線。

問題：IP 不穩定，也不易讀。

修正方向：用自訂 network 與 container name / service name。

### 一句話總結

Docker network 的重點不是背指令，而是讓服務用穩定名稱互相找到。

## Day 17：Docker 中的 DNS

### 核心概念

Docker 會替同一個 network 裡的 container 提供 DNS。Compose 會自動建立 network，並讓 service name 可解析。

### C# 設定範例

```json
{
  "ConnectionStrings": {
    "Default": "Host=db;Port=5432;Database=demo;Username=postgres;Password=postgres"
  }
}
```

### 負面例子 / 錯誤用法

錯誤做法：把 container IP 寫進 `appsettings.Production.json`。

問題：container 重建後 IP 可能改變。

修正方向：使用 DNS 名稱、環境變數與部署平台提供的 service discovery。

### 一句話總結

container 溝通要靠名稱，不要靠容易變動的 IP。

## Day 18：什麼是 Docker Image？

### 核心概念

image 是唯讀模板，包含檔案系統與啟動設定。C# image 通常分成 SDK image 與 runtime image。

### C# image 選擇

| Image | 用途 |
| --- | --- |
| `mcr.microsoft.com/dotnet/sdk:10.0` | restore、build、publish |
| `mcr.microsoft.com/dotnet/aspnet:10.0` | 跑 ASP.NET Core |
| `mcr.microsoft.com/dotnet/runtime:10.0` | 跑 console / worker |

### 負面例子 / 錯誤用法

錯誤做法：正式 API image 直接用 SDK image。

問題：image 變大，攻擊面增加，也帶入不必要工具。

修正方向：multi-stage build，正式階段只用 runtime / aspnet image。

### 一句話總結

建置需要 SDK，執行通常只需要 runtime。

## Day 19：Docker Image 的標籤

### 核心概念

tag 是 image 的版本標籤，例如 `my-api:1.0.0`、`my-api:dev`。

### C# 建議

```powershell
docker build -t my-api:dev .
docker tag my-api:dev ghcr.io/your-name/my-api:2026.05.03
```

### 負面例子 / 錯誤用法

錯誤做法：正式部署永遠用 `latest`。

問題：很難追蹤目前跑哪個版本，也不容易 rollback。

修正方向：使用 git SHA、版本號、日期 tag。

### 一句話總結

tag 是部署可追蹤性的基本單位，不要只依賴 `latest`。

## Day 20：Docker Image 快取的秘密

### 核心概念

Dockerfile 每一行通常形成一層 cache。變動越早，後面越多層要重跑。

### .NET 快取友善 Dockerfile

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src

# 先複製 csproj，讓 restore 可以被快取。
COPY *.csproj ./
RUN dotnet restore

# 原始碼較常變，放在 restore 後面。
COPY . ./
RUN dotnet publish -c Release -o /app/publish
```

### 負面例子 / 錯誤用法

錯誤做法：一開始就 `COPY . ./`，再 `dotnet restore`。

問題：任何 code 變動都會讓 restore cache 失效。

修正方向：先 copy `.csproj` restore，再 copy 全部原始碼。

### 一句話總結

Dockerfile 順序會影響建置速度，.NET 專案要特別保護 restore cache。

## Day 21：Docker Image 的唯讀性

### 核心概念

image 是唯讀的，container 執行時才有可寫層。這讓 image 可重現，也讓部署更穩定。

### C# 實務

log 應輸出到 stdout / stderr，讓 Docker 或平台收集，不要只寫在 container 內的檔案。

```csharp
app.Logger.LogInformation("API started at {Time}", DateTimeOffset.UtcNow);
```

### 負面例子 / 錯誤用法

錯誤做法：在 Dockerfile 內 bake 進正式密碼。

問題：image 一旦推到 registry，密碼就很難收回。

修正方向：密碼透過環境變數、secret、Key Vault 或部署平台注入。

### 一句話總結

image 應該描述程式與執行環境，不應該包含秘密或執行後資料。

## Day 22：推送映像檔至 DockerHub

### 核心流程

```powershell
docker login
docker tag my-api:dev yourname/my-api:1.0.0
docker push yourname/my-api:1.0.0
```

### C# 實務

公司專案常用 GitHub Container Registry、Azure Container Registry 或私有 registry。概念相同：build、tag、push、deploy。

### 負面例子 / 錯誤用法

錯誤做法：把 private app image 推到 public repository。

問題：原始檔、設定、套件版本與內部路徑可能外洩。

修正方向：確認 registry 權限與 repository visibility。

### 一句話總結

push image 前先確認 tag、權限與是否含敏感資訊。

## Day 23：Image 完整名稱與 Registry

### 核心概念

完整 image 名稱通常長這樣：

```text
registry-host/namespace/repository:tag
```

範例：

```text
ghcr.io/larihong/order-api:2026.05.03
mcr.microsoft.com/dotnet/aspnet:10.0
```

### 負面例子 / 錯誤用法

錯誤做法：不知道 image 來源，直接 `docker run random/image`。

問題：供應鏈風險高。

修正方向：優先使用官方 image、可信任組織 image，並固定版本 tag。

### 一句話總結

image 名稱不只是名字，也包含來源、擁有者與版本資訊。

## Day 24：Dockerfile 指令解析

### C# 常見指令

| 指令 | 用途 |
| --- | --- |
| `FROM` | 指定基礎 image |
| `WORKDIR` | 指定工作目錄 |
| `COPY` | 複製檔案到 image |
| `RUN` | build 階段執行命令 |
| `ENV` | 設定環境變數 |
| `EXPOSE` | 文件化 container port |
| `USER` | 指定執行身分 |
| `ENTRYPOINT` | container 啟動命令 |

### C# Dockerfile

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:10.0
WORKDIR /app
COPY ./publish .
USER app
EXPOSE 8080
ENTRYPOINT ["dotnet", "DockerCSharpDemo.dll"]
```

### 負面例子 / 錯誤用法

錯誤做法：把 `RUN dotnet run` 寫進 Dockerfile。

問題：`RUN` 是建置 image 時執行，不是 container 啟動時執行。

修正方向：建置用 `RUN dotnet publish`，啟動用 `ENTRYPOINT`。

### 一句話總結

Dockerfile 要分清楚 build-time 與 run-time。

## Day 25：建置 Docker Image

### 完整流程

1. 在 C# 專案根目錄建立 Dockerfile。
2. 建立 `.dockerignore`。
3. 執行 build。
4. 用 `docker image ls` 確認。
5. 用 `docker run` 驗證。

```powershell
docker build -t docker-csharp-demo:dev .
docker image ls docker-csharp-demo
docker run --rm -p 8080:8080 docker-csharp-demo:dev
```

### 負面例子 / 錯誤用法

錯誤做法：build 完沒有實際 run，也沒有打 endpoint。

問題：image 存在不代表 app 能啟動。

修正方向：每次 build 後至少驗證 `/health`。

### 一句話總結

image build 成功只是第一關，container 能回應才算驗證完成。

## Day 26：建置 Image 的快取機制

### C# 專案的快取重點

`bin/`、`obj/`、`.git/`、測試報告、IDE 暫存都不該進 build context。

### `.dockerignore`

```gitignore
bin/
obj/
.git/
.vs/
.vscode/
TestResults/
*.user
*.suo
```

### 負面例子 / 錯誤用法

錯誤做法：沒有 `.dockerignore`，把整個 repo 都送進 Docker build context。

問題：build 慢、cache 容易失效，也可能把不該進 image 的檔案帶進去。

修正方向：每個可 containerize 的專案都放 `.dockerignore`。

### 一句話總結

快取不是只靠 Docker，還要靠乾淨的 build context。

## Day 27：多階段建置 Docker Image

### 為什麼需要

C# build 需要 SDK，但 production 只需要 runtime。multi-stage build 可以讓最後 image 更小、更乾淨。

### 標準範例

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY *.csproj ./
RUN dotnet restore
COPY . ./
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS final
WORKDIR /app
COPY --from=build /app/publish .
USER app
EXPOSE 8080
ENTRYPOINT ["dotnet", "DockerCSharpDemo.dll"]
```

### 負面例子 / 錯誤用法

錯誤做法：把 source code、SDK、NuGet cache 全留在 final image。

問題：image 大、部署慢、暴露更多資訊。

修正方向：用 multi-stage，只 copy publish 結果。

### 一句話總結

multi-stage build 是 .NET container 化的基本功。

## Day 28：.dockerignore 該怎麼使用？

### C# 建議內容

```gitignore
**/bin/
**/obj/
.git/
.github/
.vs/
.idea/
*.md
Dockerfile*
docker-compose*.yml
```

是否忽略 `*.md`、`Dockerfile`、Compose 檔要看情境；如果應用程式 runtime 不需要它們，可以排除。

### 負面例子 / 錯誤用法

錯誤做法：把 `.env` 送進 build context。

問題：即使 Dockerfile 沒 copy，也增加秘密被誤用的風險。

修正方向：`.env` 加入 `.dockerignore`，正式秘密不要放在 repo。

### 一句話總結

`.dockerignore` 是 image 乾淨度與安全性的第一道門。

## Day 29：介紹 Docker Volume

### 核心概念

container 可刪可重建，但資料庫資料不能跟著消失。volume 是讓資料留在 container 生命週期之外的方式。

### C# + PostgreSQL 場景

API container 可以重建，PostgreSQL container 也可以重建，但 `pg-data` volume 會保存資料。

```yaml
volumes:
  pg-data:
```

### 負面例子 / 錯誤用法

錯誤做法：把 SQLite 或上傳檔案存在 container 內部路徑。

問題：container 重建後資料消失。

修正方向：使用 named volume、外部資料庫、object storage。

### 一句話總結

container 可以短暫，資料必須有自己的生命週期。

## Day 30：實際使用 Docker Volume

### 完整 Compose 範例

```yaml
services:
  api:
    build: .
    ports:
      - "8080:8080"
    environment:
      ConnectionStrings__Default: "Host=db;Port=5432;Database=demo;Username=postgres;Password=postgres"
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: demo
    volumes:
      - pg-data:/var/lib/postgresql/data

volumes:
  pg-data:
```

### 操作流程

1. 建立 `compose.yml`。
2. 執行 `docker compose up --build`。
3. API 連到 `db` service。
4. 停掉服務：`docker compose down`。
5. 再啟動：`docker compose up`。
6. 確認資料仍存在。

### 負面例子 / 錯誤用法

錯誤做法：用 `docker compose down -v` 當一般關閉指令。

問題：`-v` 會刪除 volume，資料庫資料會消失。

修正方向：日常關閉用 `docker compose down`；只有確定要重置資料時才加 `-v`。

### 一句話總結

Volume 讓你的 C# app 可以放心重建 container，而不把資料一起丟掉。

---

## 給 C# 初學者的 7 天實作路線

1. Day 1：建立 minimal API，理解本機能跑是因為裝了 .NET SDK。
2. Day 2：安裝 Docker，跑 `hello-world` 與 .NET sample image。
3. Day 3：替 minimal API 寫 Dockerfile，build image。
4. Day 4：用 `docker run -p 8080:8080` 跑 API，打 `/health`。
5. Day 5：加入 `compose.yml` 與 PostgreSQL。
6. Day 6：把 connection string 改成環境變數。
7. Day 7：加入 `.dockerignore`、multi-stage build、非 root `USER app`。

## 一個最小作品題目：容器化 Todo API

功能：

- `GET /todos`：查詢 todo。
- `POST /todos`：新增 todo。
- PostgreSQL 保存資料。
- API 和 DB 用 Docker Compose 啟動。
- API image 使用 multi-stage Dockerfile。

你會練到：

- Dockerfile。
- image build cache。
- ASP.NET Core 8080 port。
- Compose service DNS。
- environment variables。
- volume 持久化。

## 最終總結

原系列最重要的價值是建立 Docker 的基礎心智模型：image 是模板、container 是執行個體、network 讓服務互通、volume 保存資料、registry 用來分享 image。2026 年以 C# 學 Docker，建議直接用 ASP.NET Core minimal API、.NET 10 / .NET 8 官方 image、multi-stage build、Docker Compose 和非 root container 當主線。你不需要一開始就學 Kubernetes；先把「本機 C# API + 資料庫 + Docker Compose」跑穩，Docker 的大部分日常價值就會開始出現。
