# .NET Core Web API + Vue 3 全端網站應用程式：30 天教學筆記 2026 更新整理

整理日期：2026-05-05

這份筆記把後端的 ASP.NET Core Web API 和前端 Vue 3 放在同一條學習路線裡。目標不是背語法，而是讓 junior developer 能看懂「瀏覽器、Vue、API、資料庫、驗證、部署」怎麼接成一個可維護的網站應用程式。

本篇採用 `article-link-explainer` 的文章結構：每天都有固定的學習目標、名詞對照、實作步驟、實作骨架、踩雷提醒、作業、Junior 提醒與一句話收斂。

## 2026 技術基準

| 範圍 | 建議版本 / 工具 | 說明 |
| --- | --- | --- |
| 後端 | .NET 10 LTS、ASP.NET Core Web API | 以 Controller API 為主，補 Minimal API 概念 |
| ORM | EF Core | 以 SQLite 或 SQL Server LocalDB 練習，部署時可換 Azure SQL / PostgreSQL |
| 前端 | Vue 3、Vite、TypeScript | 使用 `<script setup lang="ts">`、Vue Router、Pinia |
| HTTP client | `fetch` 或 Axios | 本筆記用 service 層包裝，避免元件直接散落 request |
| 驗證 | ASP.NET Core validation + Vue form validation | 先學 DTO 驗證，再補 token 權限 |
| 測試 | xUnit / Vitest | 後端測 API 邏輯，前端測 store / component 行為 |
| 部署 | API 與 SPA 分開部署 | 後端 Web API，前端 build 後放靜態網站或 CDN |

參考文件：

- .NET releases and support: <https://learn.microsoft.com/en-us/dotnet/core/releases-and-support>
- ASP.NET Core Web API: <https://learn.microsoft.com/en-us/aspnet/core/web-api/>
- Vue Quick Start: <https://vuejs.org/guide/quick-start.html>
- Vue TypeScript Guide: <https://vuejs.org/guide/typescript/overview.html>
- Pinia: <https://pinia.vuejs.org/>

## 主線產品：ProductHub

30 天會一起完成一個真的可以展示的 `ProductHub` 商品營運後台。它不是 Todo List，而是工作上很常見的「內部管理系統」類型：前端是 Vue SPA，後端是 ASP.NET Core Web API，資料透過 EF Core 存進資料庫，最後可以 build、測試、部署、交付文件。

產品定位：

- 使用者：營運人員、商品管理員、主管
- 使用場景：管理商品資料、查看庫存、篩選上架狀態、維護商品圖片、用角色限制高風險操作
- 最終成果：一個可登入、可管理商品、可搜尋分頁、可部署的全端作品
- 技術練習重點：不是只把 API 接起來，而是練工作上常用的分層、DTO、Service、Store、環境設定、測試與部署

功能範圍：

- 儀表板：商品總數、低庫存商品、上架 / 下架統計
- 商品管理：商品清單、商品明細、建立、編輯、刪除
- 分類管理：分類清單、分類選擇、商品依分類篩選
- 庫存管理：庫存數量、低庫存提示、啟用 / 停用商品
- 關鍵字搜尋、分頁、排序
- 前端 loading / empty / error 狀態
- 後端 DTO、Service、Repository、EF Core migration
- CORS、環境設定、錯誤處理、log
- JWT 登入、授權保護 API
- API 測試、前端測試、部署檢查清單

MVP 驗收標準：

```text
1. 使用者可以登入後台
2. 使用者可以看到 dashboard 統計
3. 使用者可以查詢商品清單，並用 keyword/category/status 篩選
4. 使用者可以新增、編輯、刪除商品
5. 使用者可以查看商品明細
6. Admin 才能刪除商品
7. 表單錯誤會顯示在欄位旁
8. API 錯誤會回傳一致格式
9. 前後端都能 build
10. README 寫清楚本機啟動與 demo 流程
```

建議資料流：

```text
瀏覽器
  -> Vue Router
  -> View
  -> Component
  -> Pinia Store
  -> API Service
  -> ASP.NET Core Controller
  -> Application Service
  -> EF Core DbContext
  -> Database
```

## 工作常用專案結構建議

工作上常見的寫法不會把所有後端程式都塞在一個資料夾，也不會讓 Vue 元件到處直接打 API。這份筆記採用「後端依責任分專案、前端依功能分資料夾」的形式。

後端採用接近 Clean Architecture / Layered Architecture 的結構，但保持 junior 能消化的程度：

```text
ProductHub/
  backend/
    ProductHub.sln
    src/
      ProductHub.Api/
        Controllers/
        Middleware/
        Program.cs
        appsettings.json
      ProductHub.Application/
        Products/
          Dtos/
          Queries/
          Commands/
          Services/
        Categories/
        Auth/
        Common/
      ProductHub.Domain/
        Entities/
        Enums/
        ValueObjects/
      ProductHub.Infrastructure/
        Data/
        Repositories/
        Auth/
        Migrations/
    tests/
      ProductHub.Application.Tests/
      ProductHub.Api.Tests/
  frontend/
    product-hub-web/
      src/
        app/
          router/
          providers/
        shared/
          api/
          components/
          composables/
          types/
          utils/
        features/
          auth/
            api/
            components/
            stores/
            views/
            types/
          dashboard/
          products/
            api/
            components/
            stores/
            views/
            types/
          categories/
        main.ts
        App.vue
```

| 層級 | 後端位置 | 前端位置 | 責任 |
| --- | --- | --- | --- |
| API 入口 | `ProductHub.Api/Controllers` | `features/*/api` | 接 request、回 response、封裝 HTTP |
| Use Case | `ProductHub.Application/*` | `features/*/stores` | 實作使用者情境與狀態流程 |
| 核心模型 | `ProductHub.Domain/Entities` | `features/*/types` | 定義產品核心資料 |
| 外部資源 | `ProductHub.Infrastructure` | `shared/api` | DB、JWT、repository、HTTP client |
| 共用 UI | 不負責 UI | `shared/components` | Button、Modal、Table、FormField |
| 功能 UI | 不負責 UI | `features/*/views` / `components` | 單一功能的頁面與元件 |
| 設定 | `appsettings.*.json` | `.env.*` | API URL、連線字串、環境差異 |

### 後端依賴方向

```text
ProductHub.Api
  -> ProductHub.Application
  -> ProductHub.Domain

ProductHub.Infrastructure
  -> ProductHub.Application
  -> ProductHub.Domain
```

重點是 `Domain` 不知道資料庫，`Application` 不知道 Controller，`Api` 只處理 HTTP，`Infrastructure` 負責真的去接 DB / JWT / 外部服務。

### 前端依賴方向

```text
features/products/views
  -> features/products/stores
  -> features/products/api
  -> shared/api

features/products/components
  -> shared/components
```

重點是 page 不直接散落 `fetch()`，而是透過 feature api / store；共用元件不依賴特定業務資料。

## 30 天產品增量路線圖

| Day | 主題 | 可驗收產品增量 |
| --- | --- | --- |
| 1 | 產品需求與架構 | `docs/product-spec.md`、repo 結構、功能地圖 |
| 2 | 後端 solution 分層 | `Api/Application/Domain/Infrastructure` 可 build |
| 3 | 前端 feature-based 架構 | `features/auth/products/dashboard` 可啟動 |
| 4 | Health Check 與共用 HTTP client | 前端顯示 API 連線狀態 |
| 5 | 商品 Domain / DTO / Type | 商品資料契約完成 |
| 6 | EF Core、DbContext、Seed | 商品與分類 seed data 可查 |
| 7 | 商品清單 API | `GET /api/products` 支援基本查詢 |
| 8 | 商品清單頁 | 後台 table 顯示 loading / empty / error |
| 9 | 商品明細 | 明細 API + 明細頁可用 |
| 10 | 商品新增 | 新增 API + 新增表單可用 |
| 11 | 商品編輯 | 編輯 API + 表單復用可用 |
| 12 | 商品刪除 / 停用 | Admin 操作入口與確認流程 |
| 13 | 後端驗證與錯誤格式 | 統一 validation problem |
| 14 | 前端表單驗證 | 欄位錯誤、送出狀態、API 錯誤對應 |
| 15 | CORS 與環境設定 | dev/prod API URL 與 CORS policy |
| 16 | Pinia 商品 Store | 清單、明細、CRUD 狀態集中 |
| 17 | 搜尋、分類、排序、分頁 | 商品後台接近工作用清單 |
| 18 | Composable 與共用元件 | `useAsyncState`、`DataTable`、`ConfirmDialog` |
| 19 | 後端 Service / Repository | Controller 變薄、use case 清楚 |
| 20 | 錯誤處理與 logging | middleware / problem details / log |
| 21 | 登入 API 與 JWT | `POST /api/auth/login` 可取得 token |
| 22 | 登入頁與 route guard | 後台頁面需登入 |
| 23 | 角色授權 | Admin 才能刪除，Staff 只能編輯 |
| 24 | Dashboard | 統計卡片、低庫存列表、快速入口 |
| 25 | 商品圖片 | 圖片 URL 或 upload 欄位與預覽 |
| 26 | 後端測試 | Application service / API 測試 |
| 27 | 前端測試 | Store / component / form validation 測試 |
| 28 | Build 與部署 | API release、Vue production build、部署文件 |
| 29 | README 與 Demo Script | 可交付作品文件完成 |
| 30 | 產品驗收 | 依 demo checklist 完成全流程展示 |

---

## Day 1：全端架構與開發環境

### 今天要學什麼

今天先建立全端專案的腦中地圖：Vue 負責瀏覽器畫面，ASP.NET Core Web API 負責資料與商業規則，資料庫負責保存狀態。

### 為什麼重要

很多 junior 卡住不是因為語法，而是不知道「這段 code 應該放前端、後端還是資料庫」。先把責任邊界看懂，後面 debug 才不會亂挖。

### 關鍵名詞對照

| 名詞 | 白話說明 | 在專案的位置 |
| --- | --- | --- |
| SPA | 瀏覽器中的前端應用 | Vue |
| Web API | 提供 JSON 資料的後端 | ASP.NET Core |
| DTO | API 收送資料的格式 | `Dtos/`、`types/` |
| Entity | 資料庫對應的物件 | `Entities/` |
| Store | 前端共用狀態 | Pinia |

### 實作步驟與程式骨架

1. 建立 `ProductHub/backend` 與 `ProductHub/frontend`。
2. 確認 `.NET SDK`、Node.js、Git 版本。
3. 決定 API base URL，例如 `https://localhost:7001/api`。
4. 決定前端 dev URL，例如 `http://localhost:5173`。

```powershell
dotnet --version
node --version
npm --version
git --version
```

### 常見踩雷 / 排查

- `dotnet` 找不到：先安裝 .NET SDK，不是只裝 runtime。
- `npm` 找不到：確認 Node.js LTS 是否安裝完成。
- API 與 Vue port 混淆：前端與後端通常是兩個不同 server。

### 作業

建立 `ProductHub` 資料夾，寫一份 `docs/architecture.md`，畫出瀏覽器到資料庫的流程。

### Junior 提醒

全端不是把所有東西塞在一起，而是知道每一層該負責什麼。

### 一句話收斂

今天先看懂路線圖，明天才開始動手蓋後端。

---

## Day 2：建立 ASP.NET Core Web API

### 今天要學什麼

建立第一個 ASP.NET Core Web API 專案，理解 `Program.cs`、Controller、route、Swagger 的基本角色。

### 為什麼重要

Web API 是前端取得資料的入口。Vue 不應該直接碰資料庫，而是透過 HTTP request 跟 API 溝通。

### 關鍵名詞對照

| 名詞 | 白話說明 | 例子 |
| --- | --- | --- |
| Controller | API 的入口類別 | `ProductsController` |
| Action | API endpoint 的方法 | `GetProducts()` |
| Route | URL 對應規則 | `/api/products` |
| Swagger | API 文件與測試頁 | `/swagger` |

### 實作步驟與程式骨架

```powershell
cd ProductHub/backend
dotnet new sln -n ProductHub
mkdir src
mkdir tests

dotnet new webapi -n ProductHub.Api -o src/ProductHub.Api
dotnet new classlib -n ProductHub.Application -o src/ProductHub.Application
dotnet new classlib -n ProductHub.Domain -o src/ProductHub.Domain
dotnet new classlib -n ProductHub.Infrastructure -o src/ProductHub.Infrastructure

dotnet sln add src/ProductHub.Api/ProductHub.Api.csproj
dotnet sln add src/ProductHub.Application/ProductHub.Application.csproj
dotnet sln add src/ProductHub.Domain/ProductHub.Domain.csproj
dotnet sln add src/ProductHub.Infrastructure/ProductHub.Infrastructure.csproj

dotnet add src/ProductHub.Api/ProductHub.Api.csproj reference src/ProductHub.Application/ProductHub.Application.csproj
dotnet add src/ProductHub.Api/ProductHub.Api.csproj reference src/ProductHub.Infrastructure/ProductHub.Infrastructure.csproj
dotnet add src/ProductHub.Application/ProductHub.Application.csproj reference src/ProductHub.Domain/ProductHub.Domain.csproj
dotnet add src/ProductHub.Infrastructure/ProductHub.Infrastructure.csproj reference src/ProductHub.Application/ProductHub.Application.csproj

cd src/ProductHub.Api
dotnet run
```

專案責任先這樣記：

| 專案 | 責任 | 不該做的事 |
| --- | --- | --- |
| `ProductHub.Api` | Controller、middleware、DI、HTTP 設定 | 不寫大量商業邏輯 |
| `ProductHub.Application` | 商品 use case、DTO、service interface | 不直接依賴 Web API |
| `ProductHub.Domain` | Entity、enum、核心規則 | 不知道 EF Core、Controller |
| `ProductHub.Infrastructure` | EF Core、repository、JWT、外部服務 | 不處理 HTTP response |

新增健康檢查 API：

```csharp
// Controllers/HealthController.cs
using Microsoft.AspNetCore.Mvc;

namespace ProductHub.Api.Controllers;

[ApiController]
[Route("api/[controller]")]
public sealed class HealthController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new { status = "ok", app = "ProductHub.Api" });
    }
}
```

### 常見踩雷 / 排查

- 找不到 Swagger：確認開發環境有啟用 Swagger middleware。
- 404：檢查 route 是 `/api/health` 還是 `/health`。
- HTTPS 憑證警告：開發階段可先信任 dev certificate。

### 作業

啟動 API，從 Swagger 或瀏覽器打開 `/api/health`。

### Junior 提醒

Controller 的第一個責任是處理 HTTP，不是寫一大坨商業邏輯。

### 一句話收斂

後端今天先能回答「我活著」。

---

## Day 3：建立 Vue 3 Vite 專案

### 今天要學什麼

建立 Vue 3 + Vite + TypeScript 專案，理解 `main.ts`、`App.vue`、View 與 Component。

### 為什麼重要

Vue 是使用者實際操作的地方。前端要把 API 回來的 JSON 轉成可讀、可操作、可回饋的畫面。

### 關鍵名詞對照

| 名詞 | 白話說明 | 位置 |
| --- | --- | --- |
| Vite | 前端開發與 build 工具 | `vite.config.ts` |
| SFC | `.vue` 單檔元件 | `App.vue` |
| View | 對應頁面的元件 | `views/` |
| Component | 可重用 UI 區塊 | `components/` |

### 實作步驟與程式骨架

```powershell
cd ProductHub/frontend
npm create vue@latest product-hub-web
cd product-hub-web
npm install
npm run dev
```

接著先整理成工作上比較常見的 feature-based 結構：

```text
src/
  app/
    router/
    providers/
  shared/
    api/
      httpClient.ts
    components/
      BaseButton.vue
      ConfirmDialog.vue
      DataTable.vue
      FormField.vue
    composables/
      useAsyncState.ts
    types/
  features/
    auth/
      api/
      stores/
      views/
      types/
    dashboard/
      api/
      components/
      views/
    products/
      api/
      components/
      stores/
      views/
      types/
    categories/
      api/
      stores/
      views/
      types/
  main.ts
  App.vue
```

這個結構的判斷方式：

| 檔案類型 | 放哪裡 | 判斷方式 |
| --- | --- | --- |
| 只屬於商品功能 | `features/products/*` | 離開商品模組就不會用到 |
| 登入、權限 | `features/auth/*` | 跟使用者身份有關 |
| 多個功能都會用 | `shared/*` | Button、Modal、HTTP client、通用 composable |
| App 啟動設定 | `app/*` | router、provider、全域註冊 |

建議選項：

- TypeScript：Yes
- Vue Router：Yes
- Pinia：Yes
- Vitest：Yes
- ESLint / Prettier：依團隊習慣

### 常見踩雷 / 排查

- `npm create vue@latest` 失敗：多半是網路或 Node 版本問題。
- 畫面空白：看瀏覽器 console 與 terminal error。
- `.vue` 沒有語法提示：VS Code 建議安裝 `Vue - Official`。

### 作業

新增 `ProductListView.vue`，讓首頁先顯示「商品管理」。

### Junior 提醒

先讓 app 跑起來，再談架構。跑不起來的架構沒有意義。

### 一句話收斂

今天前端先有一個可以打開的殼。

---

## Day 4：第一支 API 與第一個畫面

### 今天要學什麼

讓 Vue 呼叫 ASP.NET Core 的 `/api/health`，完成第一條前後端連線。

### 為什麼重要

全端開發最核心的肌肉是 request / response。只要你能穩定看懂每一次 HTTP 往返，debug 會輕鬆很多。

### 關鍵名詞對照

| 名詞 | 白話說明 | 例子 |
| --- | --- | --- |
| HTTP GET | 向 API 讀資料 | `GET /api/health` |
| Response | API 回傳結果 | `{ status: "ok" }` |
| API Service | 前端封裝 request 的檔案 | `services/healthApi.ts` |

### 實作步驟與程式骨架

```ts
// src/services/healthApi.ts
export interface HealthResponse {
  status: string
  app: string
}

export async function getHealth(): Promise<HealthResponse> {
  const response = await fetch(`${import.meta.env.VITE_API_BASE_URL}/health`)

  if (!response.ok) {
    throw new Error(`Health check failed: ${response.status}`)
  }

  return response.json()
}
```

```env
# .env.development
VITE_API_BASE_URL=https://localhost:7001/api
```

### 常見踩雷 / 排查

- CORS error：瀏覽器擋跨來源，後端要允許 Vue dev server。
- `Failed to fetch`：API 沒啟動、URL 錯、HTTPS 憑證不信任。
- `.env` 改完沒生效：重啟 Vite dev server。

### 作業

在首頁顯示 API 狀態，並處理 loading 與 error。

### Junior 提醒

前端錯誤不一定是 Vue 錯，也可能是 API 沒開、URL 錯或 CORS。

### 一句話收斂

今天讓瀏覽器真的跟後端說上話。

---

## Day 5：DTO、Entity、Response Model

### 今天要學什麼

定義商品的資料長相，分清楚 Entity、Request DTO、Response DTO、前端 TypeScript type。

### 為什麼重要

Entity 是資料庫內部模型，不應該直接裸露給前端。API contract 穩定，前後端才好協作。

### 關鍵名詞對照

| 名詞 | 責任 | 範例 |
| --- | --- | --- |
| Entity | 資料庫保存格式 | `Product` |
| Request DTO | 前端送進 API 的格式 | `CreateProductRequest` |
| Response DTO | API 回給前端的格式 | `ProductResponse` |
| TypeScript Type | 前端使用的資料型別 | `Product` |

### 實作步驟與程式骨架

```csharp
// Entities/Product.cs
namespace ProductHub.Api.Entities;

public sealed class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int Stock { get; set; }
    public bool IsActive { get; set; } = true;
    public DateTimeOffset CreatedAt { get; set; } = DateTimeOffset.UtcNow;
}
```

```csharp
// Dtos/ProductDtos.cs
namespace ProductHub.Api.Dtos;

public sealed record ProductResponse(
    int Id,
    string Name,
    decimal Price,
    int Stock,
    bool IsActive);

public sealed record CreateProductRequest(
    string Name,
    decimal Price,
    int Stock);
```

```ts
// src/types/product.ts
export interface Product {
  id: number
  name: string
  price: number
  stock: number
  isActive: boolean
}
```

### 常見踩雷 / 排查

- 後端屬性是 `Name`，前端拿到常是 `name`：JSON 命名策略會轉 camelCase。
- decimal 到 JavaScript 會變 number：金額精度高的系統要更小心。
- Entity 直接回傳：容易把不該公開的欄位也送出去。

### 作業

補上 `UpdateProductRequest`，思考哪些欄位可以被修改。

### Junior 提醒

DTO 是 API 的合約，不是多此一舉。

### 一句話收斂

今天先把資料講清楚，後面 CRUD 才不會歪。

---

## Day 6：EF Core 與資料庫

### 今天要學什麼

使用 EF Core 建立 DbContext、migration 與初始資料。

### 為什麼重要

API 的資料不能永遠寫死在 memory 裡。資料庫讓商品可以被長期保存、查詢與修改。

### 關鍵名詞對照

| 名詞 | 白話說明 |
| --- | --- |
| DbContext | EF Core 與資料庫溝通的入口 |
| DbSet | 某一張資料表的集合 |
| Migration | 資料庫結構版本 |
| Seed Data | 開發用初始資料 |

### 實作步驟與程式骨架

```csharp
// Data/AppDbContext.cs
using Microsoft.EntityFrameworkCore;
using ProductHub.Api.Entities;

namespace ProductHub.Api.Data;

public sealed class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }

    public DbSet<Product> Products => Set<Product>();
}
```

```csharp
// Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));
```

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=producthub.db"
  }
}
```

```powershell
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet ef migrations add InitialCreate
dotnet ef database update
```

### 常見踩雷 / 排查

- `dotnet ef` 找不到：安裝 `dotnet-ef` tool。
- migration 失敗：確認 DbContext 有註冊，package 有裝。
- 連線字串 commit 到 repo：開發可放 SQLite，正式密碼要放 secret store。

### 作業

新增三筆商品 seed data，讓清單 API 可以讀到資料。

### Junior 提醒

Migration 是資料庫版控，不是錯了就亂刪檔。

### 一句話收斂

今天讓資料有地方住。

---

## Day 7：商品清單 API

### 今天要學什麼

建立 `GET /api/products`，從資料庫讀商品並轉成 response DTO。

### 為什麼重要

清單 API 是大多數管理系統的第一個核心功能，也是搜尋、分頁、排序的基礎。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| `IActionResult` | API action 的回傳型別 |
| `Ok()` | 回傳 HTTP 200 |
| `Select()` | 將 Entity 投影成 DTO |
| `ToListAsync()` | 非同步查詢資料庫 |

### 實作步驟與程式骨架

```csharp
// Controllers/ProductsController.cs
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using ProductHub.Api.Data;
using ProductHub.Api.Dtos;

namespace ProductHub.Api.Controllers;

[ApiController]
[Route("api/[controller]")]
public sealed class ProductsController : ControllerBase
{
    private readonly AppDbContext _db;

    public ProductsController(AppDbContext db)
    {
        _db = db;
    }

    [HttpGet]
    public async Task<ActionResult<IReadOnlyList<ProductResponse>>> GetProducts()
    {
        var products = await _db.Products
            .OrderByDescending(product => product.Id)
            .Select(product => new ProductResponse(
                product.Id,
                product.Name,
                product.Price,
                product.Stock,
                product.IsActive))
            .ToListAsync();

        return Ok(products);
    }
}
```

### 常見踩雷 / 排查

- 回傳 Entity：先投影成 DTO。
- 忘記 `await`：API 可能回傳 Task 物件或編譯失敗。
- 資料庫沒有資料：確認 seed 或手動新增。

### 作業

用 Swagger 測 `GET /api/products`，確認回傳 JSON array。

### Junior 提醒

API 不是只要能回資料就好，也要回「前端真的需要的資料」。

### 一句話收斂

今天完成第一個真正讀資料的 API。

---

## Day 8：Vue 商品清單頁

### 今天要學什麼

前端呼叫商品清單 API，顯示 loading、empty、error、data 四種狀態。

### 為什麼重要

真實 UI 不只有成功狀態。使用者等待、失敗、沒有資料時都需要清楚回饋。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| loading | request 進行中 |
| error | request 失敗 |
| empty | request 成功但沒有資料 |
| data | request 成功且有資料 |

### 實作步驟與程式骨架

```ts
// src/services/productApi.ts
import type { Product } from '@/types/product'

const baseUrl = import.meta.env.VITE_API_BASE_URL

export async function listProducts(): Promise<Product[]> {
  const response = await fetch(`${baseUrl}/products`)

  if (!response.ok) {
    throw new Error(`Load products failed: ${response.status}`)
  }

  return response.json()
}
```

```vue
<!-- src/views/ProductListView.vue -->
<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { listProducts } from '@/services/productApi'
import type { Product } from '@/types/product'

const products = ref<Product[]>([])
const loading = ref(false)
const error = ref<string | null>(null)

async function loadProducts() {
  loading.value = true
  error.value = null

  try {
    products.value = await listProducts()
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Unknown error'
  } finally {
    loading.value = false
  }
}

onMounted(loadProducts)
</script>

<template>
  <main>
    <h1>商品管理</h1>

    <p v-if="loading">載入中...</p>
    <p v-else-if="error">{{ error }}</p>
    <p v-else-if="products.length === 0">目前沒有商品</p>

    <table v-else>
      <thead>
        <tr>
          <th>名稱</th>
          <th>價格</th>
          <th>庫存</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in products" :key="product.id">
          <td>{{ product.name }}</td>
          <td>{{ product.price }}</td>
          <td>{{ product.stock }}</td>
        </tr>
      </tbody>
    </table>
  </main>
</template>
```

### 常見踩雷 / 排查

- `products.map is not a function`：API 回來不是 array。
- 畫面沒有更新：確認 `ref` 使用 `.value`。
- CORS：回 Day 15 前可先暫時設定允許 dev origin。

### 作業

把 table 抽成 `ProductTable.vue`，用 props 接收商品列表。

### Junior 提醒

前端 service 負責 HTTP，component 負責呈現，兩者分開才好測。

### 一句話收斂

今天讓資料從資料庫一路走到畫面。

---

## Day 9：商品明細 API 與頁面

### 今天要學什麼

建立 `GET /api/products/{id}` 與 Vue detail route。

### 為什麼重要

明細頁會練到 route parameter、404、前端依照 URL 載入資料。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Route Parameter | URL 中的變數，例如 `{id}` |
| 404 | 找不到資源 |
| `useRoute()` | Vue Router 讀目前路由 |

### 實作步驟與程式骨架

```csharp
[HttpGet("{id:int}")]
public async Task<ActionResult<ProductResponse>> GetProduct(int id)
{
    var product = await _db.Products
        .Where(product => product.Id == id)
        .Select(product => new ProductResponse(
            product.Id,
            product.Name,
            product.Price,
            product.Stock,
            product.IsActive))
        .FirstOrDefaultAsync();

    return product is null ? NotFound() : Ok(product);
}
```

```ts
// router/index.ts
{
  path: '/products/:id',
  name: 'product-detail',
  component: () => import('@/views/ProductDetailView.vue'),
}
```

### 常見踩雷 / 排查

- id 是 string：前端 route param 需要轉成 number 或直接組 URL。
- 後端找不到時回空物件：應回 404。
- 明細頁重複載入：確認 watch route 的時機。

### 作業

商品清單每列加上「查看」連結，導到明細頁。

### Junior 提醒

找不到資料不是例外，是正常情境，要用 404 表達。

### 一句話收斂

今天讓 URL 可以代表某一筆資料。

---

## Day 10：建立商品 API 與表單

### 今天要學什麼

建立 `POST /api/products`，前端做新增表單。

### 為什麼重要

建立資料會練到 request body、DTO、validation、送出狀態與成功後導頁。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| POST | 建立資料 |
| Request Body | 前端送出的 JSON |
| 201 Created | 建立成功的 HTTP 狀態 |

### 實作步驟與程式骨架

```csharp
[HttpPost]
public async Task<ActionResult<ProductResponse>> CreateProduct(CreateProductRequest request)
{
    var product = new Product
    {
        Name = request.Name.Trim(),
        Price = request.Price,
        Stock = request.Stock
    };

    _db.Products.Add(product);
    await _db.SaveChangesAsync();

    var response = new ProductResponse(
        product.Id,
        product.Name,
        product.Price,
        product.Stock,
        product.IsActive);

    return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, response);
}
```

```ts
export interface CreateProductRequest {
  name: string
  price: number
  stock: number
}

export async function createProduct(request: CreateProductRequest): Promise<Product> {
  const response = await fetch(`${baseUrl}/products`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(request),
  })

  if (!response.ok) {
    throw new Error(`Create product failed: ${response.status}`)
  }

  return response.json()
}
```

### 常見踩雷 / 排查

- 忘記 `Content-Type: application/json`：後端可能收不到 body。
- 建立成功回 200：可行，但 201 更符合語意。
- 表單重複送出：送出時停用按鈕。

### 作業

完成 `ProductForm.vue`，新增成功後導回清單頁。

### Junior 提醒

新增資料要想「送出前、送出中、成功、失敗」四個狀態。

### 一句話收斂

今天使用者第一次能改變系統資料。

---

## Day 11：編輯商品 API 與表單復用

### 今天要學什麼

建立 `PUT /api/products/{id}`，用同一個 `ProductForm.vue` 處理新增與編輯。

### 為什麼重要

表單復用會讓你開始思考 props、emits、初始值與送出事件的邊界。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| PUT | 更新整筆資源 |
| Form Model | 表單正在編輯的資料 |
| emits | 子元件通知父元件 |

### 實作步驟與程式骨架

```csharp
[HttpPut("{id:int}")]
public async Task<IActionResult> UpdateProduct(int id, UpdateProductRequest request)
{
    var product = await _db.Products.FindAsync(id);

    if (product is null)
    {
        return NotFound();
    }

    product.Name = request.Name.Trim();
    product.Price = request.Price;
    product.Stock = request.Stock;
    product.IsActive = request.IsActive;

    await _db.SaveChangesAsync();
    return NoContent();
}
```

### 常見踩雷 / 排查

- 編輯表單直接改 props：Vue 會警告，應複製成 local form state。
- `PUT` 成功後前端資料沒更新：導頁或重新載入 store。
- id 不存在：回 404。

### 作業

把新增與編輯都改用同一個 `ProductForm.vue`。

### Junior 提醒

共用元件不是把所有情境塞成一團，而是抽出真的相同的部分。

### 一句話收斂

今天讓表單從一次性程式碼變成可復用元件。

---

## Day 12：刪除商品與確認流程

### 今天要學什麼

建立 `DELETE /api/products/{id}`，前端加入刪除確認與 optimistic / reload 的取捨。

### 為什麼重要

刪除是破壞性操作。流程要避免誤刪，也要讓畫面狀態保持一致。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| DELETE | 刪除資源 |
| Confirm Dialog | 確認操作 |
| Optimistic UI | 先更新畫面，再處理失敗回復 |

### 實作步驟與程式骨架

```csharp
[HttpDelete("{id:int}")]
public async Task<IActionResult> DeleteProduct(int id)
{
    var product = await _db.Products.FindAsync(id);

    if (product is null)
    {
        return NotFound();
    }

    _db.Products.Remove(product);
    await _db.SaveChangesAsync();
    return NoContent();
}
```

### 常見踩雷 / 排查

- 刪除後清單還在：刪除成功後重新載入或從 local state 移除。
- 誤刪：至少要有確認流程。
- 真實系統可能用軟刪除：用 `IsDeleted` 比直接刪資料更安全。

### 作業

商品清單加刪除按鈕，成功後重新載入清單。

### Junior 提醒

刪除功能要比新增功能更謹慎，因為它會讓資料消失。

### 一句話收斂

今天完成 CRUD 的最後一塊。

---

## Day 13：驗證與錯誤回應

### 今天要學什麼

使用 Data Annotations 驗證 DTO，並理解 ASP.NET Core 的 validation problem response。

### 為什麼重要

後端驗證是最後防線。前端驗證能改善體驗，但不能取代後端驗證。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Validation | 檢查輸入是否合理 |
| Data Annotation | C# attribute 驗證規則 |
| Problem Details | 標準化錯誤回應格式 |

### 實作步驟與程式骨架

```csharp
using System.ComponentModel.DataAnnotations;

public sealed record CreateProductRequest(
    [Required, StringLength(100, MinimumLength = 2)] string Name,
    [Range(1, 999999)] decimal Price,
    [Range(0, 99999)] int Stock);
```

錯誤回應概念：

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Name": ["The Name field is required."]
  }
}
```

### 常見踩雷 / 排查

- 只做前端驗證：使用者可以直接打 API。
- 錯誤訊息不穩定：前端應能處理欄位錯誤與一般錯誤。
- decimal 範圍沒限制：負價格會進資料庫。

### 作業

故意送空名稱與負價格，觀察 API 回傳。

### Junior 提醒

驗證規則越靠近資料入口越好，前後端都要做，但目的不同。

### 一句話收斂

今天讓 API 學會拒絕壞資料。

---

## Day 14：前端表單驗證

### 今天要學什麼

在 Vue 表單處理 required、number range、API validation errors。

### 為什麼重要

好的表單不是只會送資料，而是能清楚告訴使用者哪裡需要修正。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Client Validation | 前端先檢查 |
| Server Validation | 後端最終檢查 |
| Field Error | 欄位層級錯誤 |

### 實作步驟與程式骨架

```ts
export interface FieldErrors {
  [field: string]: string[]
}

export function validateProductForm(form: CreateProductRequest): FieldErrors {
  const errors: FieldErrors = {}

  if (form.name.trim().length < 2) {
    errors.name = ['商品名稱至少 2 個字']
  }

  if (form.price <= 0) {
    errors.price = ['價格必須大於 0']
  }

  if (form.stock < 0) {
    errors.stock = ['庫存不可小於 0']
  }

  return errors
}
```

### 常見踩雷 / 排查

- 錯誤訊息只用 alert：使用者不知道哪個欄位錯。
- 前端規則跟後端不一致：以後端為準。
- number input 拿到 string：注意轉型。

### 作業

讓 `ProductForm.vue` 顯示欄位錯誤，並在送出中停用按鈕。

### Junior 提醒

表單 UX 是工程品質的一部分。

### 一句話收斂

今天讓錯誤不只是失敗，而是可修正的提示。

---

## Day 15：CORS 與環境變數

### 今天要學什麼

設定 ASP.NET Core CORS，並用 `.env` 管理 Vue API base URL。

### 為什麼重要

前後端分開開發時一定會遇到跨來源。正式環境也不能亂開 `AllowAnyOrigin`。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Origin | protocol + domain + port |
| CORS | 瀏覽器跨來源存取規則 |
| Preflight | 瀏覽器先送 OPTIONS 檢查 |

### 實作步驟與程式骨架

```csharp
// Program.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("Frontend", policy =>
    {
        policy
            .WithOrigins("http://localhost:5173")
            .AllowAnyHeader()
            .AllowAnyMethod();
    });
});

var app = builder.Build();

app.UseCors("Frontend");
```

```env
VITE_API_BASE_URL=https://localhost:7001/api
```

### 常見踩雷 / 排查

- CORS 是瀏覽器限制，不是 API 不能被呼叫。
- middleware 順序錯：CORS 要放在合適的位置。
- 正式環境開 `AllowAnyOrigin`：安全風險高。

### 作業

把 dev API URL 移到 `.env.development`，不要寫死在 service。

### Junior 提醒

CORS 錯誤要看瀏覽器 console，不是只看 API terminal。

### 一句話收斂

今天讓前後端跨來源溝通變成可控設定。

---

## Day 16：Pinia Store

### 今天要學什麼

用 Pinia 管理商品清單、目前商品、loading、error。

### 為什麼重要

當多個頁面都需要商品資料時，把狀態集中管理會比每個 view 自己打 API 更穩。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Store | 共用狀態容器 |
| State | 資料 |
| Action | 改變狀態的非同步流程 |
| Getter | 衍生資料 |

### 實作步驟與程式骨架

```ts
// src/stores/productStore.ts
import { defineStore } from 'pinia'
import { listProducts } from '@/services/productApi'
import type { Product } from '@/types/product'

export const useProductStore = defineStore('product', {
  state: () => ({
    products: [] as Product[],
    loading: false,
    error: null as string | null,
  }),
  actions: {
    async loadProducts() {
      this.loading = true
      this.error = null

      try {
        this.products = await listProducts()
      } catch (err) {
        this.error = err instanceof Error ? err.message : 'Unknown error'
      } finally {
        this.loading = false
      }
    },
  },
})
```

### 常見踩雷 / 排查

- component 和 store 都各自保存一份資料：容易不同步。
- action 直接吞錯誤：UI 會不知道失敗。
- store 變成垃圾桶：只放跨頁共用狀態。

### 作業

將商品清單頁改成使用 `useProductStore()`。

### Junior 提醒

Store 是共用狀態，不是所有變數都要丟進去。

### 一句話收斂

今天讓前端資料有一個共同來源。

---

## Day 17：搜尋、排序、分頁

### 今天要學什麼

後端接收 query string，前端建立搜尋、排序與分頁 UI。

### 為什麼重要

資料一多，清單就不能一次全部載入。搜尋與分頁是管理系統的基本能力。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Query String | URL 問號後面的參數 |
| Pagination | 分頁 |
| Sort | 排序 |
| Filter | 篩選 |

### 實作步驟與程式骨架

```csharp
public sealed record ProductQuery(
    string? Keyword,
    int Page = 1,
    int PageSize = 20,
    string Sort = "id_desc");
```

```csharp
[HttpGet]
public async Task<ActionResult<IReadOnlyList<ProductResponse>>> GetProducts([FromQuery] ProductQuery query)
{
    var page = Math.Max(query.Page, 1);
    var pageSize = Math.Clamp(query.PageSize, 1, 100);

    var productsQuery = _db.Products.AsQueryable();

    if (!string.IsNullOrWhiteSpace(query.Keyword))
    {
        productsQuery = productsQuery.Where(product => product.Name.Contains(query.Keyword));
    }

    productsQuery = query.Sort == "price_asc"
        ? productsQuery.OrderBy(product => product.Price)
        : productsQuery.OrderByDescending(product => product.Id);

    var products = await productsQuery
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .Select(product => new ProductResponse(product.Id, product.Name, product.Price, product.Stock, product.IsActive))
        .ToListAsync();

    return Ok(products);
}
```

### 常見踩雷 / 排查

- 先 `ToListAsync()` 再分頁：會把所有資料拉到記憶體。
- page 小於 1：要修正或回錯誤。
- 前端搜尋每打一字都打 API：需要 debounce。

### 作業

前端加 keyword、sort、page 控制，送到 API query string。

### Junior 提醒

分頁要在資料庫層處理，不是把全部資料拿回前端再切。

### 一句話收斂

今天讓清單開始像真正的後台系統。

---

## Day 18：Composable 抽離共用邏輯

### 今天要學什麼

把 loading / error / async request、分頁狀態抽成 composable。

### 為什麼重要

當多個頁面都需要同樣的非同步流程，composable 可以讓程式碼保持清楚。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Composable | Vue 組合式邏輯函式 |
| `ref` | 可追蹤的單一狀態 |
| `computed` | 衍生狀態 |
| `watch` | 監聽變化 |

### 實作步驟與程式骨架

```ts
// src/composables/useAsyncState.ts
import { ref } from 'vue'

export function useAsyncState() {
  const loading = ref(false)
  const error = ref<string | null>(null)

  async function run<T>(task: () => Promise<T>): Promise<T | null> {
    loading.value = true
    error.value = null

    try {
      return await task()
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'Unknown error'
      return null
    } finally {
      loading.value = false
    }
  }

  return { loading, error, run }
}
```

### 常見踩雷 / 排查

- composable 偷偷改全域狀態：命名與責任要清楚。
- 抽太早：重複出現兩三次再抽比較穩。
- 沒有回傳 error：UI 失去回饋。

### 作業

將清單頁的 loading/error 流程改成使用 `useAsyncState()`。

### Junior 提醒

Composable 是抽邏輯，不是抽 HTML。

### 一句話收斂

今天讓重複的前端流程變成可重用工具。

---

## Day 19：後端 Service / Repository

### 今天要學什麼

把 Controller 中的商品邏輯移到 Service，讓 Controller 只處理 HTTP。

### 為什麼重要

Controller 一旦肥大，測試、維護、重用都會變難。Service 讓商業邏輯有自己的位置。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Application Service | 處理 use case |
| Repository | 封裝資料存取 |
| DI | 由框架注入依賴 |

### 實作步驟與程式骨架

```csharp
public interface IProductService
{
    Task<IReadOnlyList<ProductResponse>> GetProductsAsync(ProductQuery query);
    Task<ProductResponse?> GetProductAsync(int id);
}
```

```csharp
builder.Services.AddScoped<IProductService, ProductService>();
```

Controller 改成：

```csharp
private readonly IProductService _productService;

public ProductsController(IProductService productService)
{
    _productService = productService;
}
```

### 常見踩雷 / 排查

- Service 又直接回 `IActionResult`：HTTP 細節留在 Controller。
- Repository 過度抽象：小專案可先 Service + DbContext。
- 忘記註冊 DI：執行時會噴無法解析 service。

### 作業

把清單、明細、建立、更新、刪除搬到 `ProductService`。

### Junior 提醒

Controller 要薄，Service 要清楚，DbContext 不要到處亂傳。

### 一句話收斂

今天讓後端開始有可維護的分層。

---

## Day 20：統一錯誤處理與 logging

### 今天要學什麼

建立全域 exception handling，並使用 logging 記錄錯誤。

### 為什麼重要

真實系統一定會出錯。重點不是永遠不錯，而是錯了能被看見、被定位、被安全回應。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Middleware | HTTP pipeline 中的處理節點 |
| Exception Handler | 統一處理未捕捉例外 |
| Log | 系統執行紀錄 |
| Problem Details | API 錯誤標準格式 |

### 實作步驟與程式骨架

```csharp
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler();
```

Service 中使用 log：

```csharp
private readonly ILogger<ProductService> _logger;

public ProductService(AppDbContext db, ILogger<ProductService> logger)
{
    _db = db;
    _logger = logger;
}
```

### 常見踩雷 / 排查

- 把 exception message 直接回給前端：可能洩漏內部資訊。
- catch 後什麼都不做：錯誤會消失。
- 前端只顯示 raw error：使用者看不懂。

### 作業

故意在 API 中丟例外，確認前端看到友善錯誤，後端 log 看得到細節。

### Junior 提醒

錯誤訊息分兩種：給使用者看的，給工程師查問題看的。

### 一句話收斂

今天讓系統出錯時不再沉默。

---

## Day 21：登入 API 與 JWT

### 今天要學什麼

建立簡化登入 API，成功後回傳 JWT token。

### 為什麼重要

管理系統通常不能讓所有人都能新增、修改、刪除。登入是權限控制的入口。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| JWT | 可被驗證的 token |
| Claim | token 中的使用者資訊 |
| Bearer Token | HTTP Authorization header 格式 |

### 實作步驟與程式骨架

```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "admin@example.com",
  "password": "P@ssw0rd!"
}
```

回應：

```json
{
  "accessToken": "<jwt>",
  "expiresIn": 3600
}
```

### 常見踩雷 / 排查

- 把密碼明文存資料庫：正式系統一定要 hash。
- JWT secret 寫在 repo：正式環境要用 secret store。
- token 過期沒處理：前端要導回登入。

### 作業

完成 `AuthController.Login()`，先用固定帳密練習 token flow。

### Junior 提醒

教學可以簡化帳密，但正式系統不能簡化安全。

### 一句話收斂

今天讓 API 開始認得使用者。

---

## Day 22：前端登入與 route guard

### 今天要學什麼

Vue 建立登入頁，保存 token，並用 route guard 保護後台頁面。

### 為什麼重要

有 token 不代表 UI 自動安全。前端要控制頁面進入流程，後端仍要保護 API。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Auth Store | 保存登入狀態 |
| Route Guard | 進入頁面前的檢查 |
| Authorization Header | `Bearer <token>` |

### 實作步驟與程式骨架

```ts
// router/index.ts
router.beforeEach((to) => {
  const auth = useAuthStore()

  if (to.meta.requiresAuth && !auth.isAuthenticated) {
    return { name: 'login', query: { redirect: to.fullPath } }
  }
})
```

API service 加 token：

```ts
headers: {
  'Content-Type': 'application/json',
  Authorization: `Bearer ${auth.accessToken}`,
}
```

### 常見踩雷 / 排查

- 只做 route guard，不保護 API：使用者仍可直接打 API。
- token 放 localStorage 有 XSS 風險：正式系統要評估安全策略。
- 登出只導頁，沒清 token：狀態會殘留。

### 作業

登入後導回原本想去的頁面，登出後清除 auth store。

### Junior 提醒

前端權限是 UX，後端權限才是安全邊界。

### 一句話收斂

今天讓前端知道誰登入了。

---

## Day 23：授權與角色

### 今天要學什麼

後端用 `[Authorize]` 與 role 限制 API，前端依角色顯示操作按鈕。

### 為什麼重要

不是所有登入者都有相同權限。角色與 policy 可以讓權限規則清楚化。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Authentication | 你是誰 |
| Authorization | 你能做什麼 |
| Role | 使用者角色 |
| Policy | 更彈性的授權規則 |

### 實作步驟與程式骨架

```csharp
[Authorize]
[HttpGet]
public async Task<ActionResult<IReadOnlyList<ProductResponse>>> GetProducts()
{
    // ...
}

[Authorize(Roles = "Admin")]
[HttpDelete("{id:int}")]
public async Task<IActionResult> DeleteProduct(int id)
{
    // ...
}
```

### 常見踩雷 / 排查

- 401 與 403 混淆：401 是未登入，403 是沒權限。
- 前端藏按鈕就以為安全：後端仍要檢查。
- token 裡沒有 role claim：授權永遠不通過。

### 作業

只有 Admin 看得到刪除按鈕，且 API 也必須驗證 Admin。

### Junior 提醒

權限規則要在後端落地，前端只是配合顯示。

### 一句話收斂

今天讓系統不只認人，也認權限。

---

## Day 24：檔案上傳或圖片 URL

### 今天要學什麼

為商品加入圖片欄位，練習圖片 URL 或 `multipart/form-data` 上傳。

### 為什麼重要

商品管理常需要圖片。圖片處理會牽涉檔案大小、格式、安全與儲存位置。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Image URL | 圖片網址 |
| Multipart Form | 檔案上傳格式 |
| Static Files | 後端提供靜態檔案 |
| Object Storage | 雲端檔案儲存 |

### 實作步驟與程式骨架

先用圖片 URL 簡化：

```csharp
public sealed record UpdateProductImageRequest(string ImageUrl);
```

```ts
export interface Product {
  id: number
  name: string
  price: number
  stock: number
  isActive: boolean
  imageUrl?: string
}
```

### 常見踩雷 / 排查

- 直接信任任意 URL：正式系統要驗證與白名單策略。
- 上傳檔案不限制大小：容易被濫用。
- 圖片路徑存本機：多台 server 部署會不同步。

### 作業

商品表單加入圖片 URL，清單顯示縮圖。

### Junior 提醒

檔案處理比字串欄位更容易出安全與部署問題。

### 一句話收斂

今天讓商品資料開始接近真實使用情境。

---

## Day 25：後端測試

### 今天要學什麼

用 xUnit 測試 `ProductService`，確認建立與驗證邏輯。

### 為什麼重要

測試可以保護你未來重構時不把 CRUD 行為弄壞。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Unit Test | 測單一邏輯 |
| Integration Test | 測 API 與資料庫整合 |
| Arrange / Act / Assert | 測試三段式 |

### 實作步驟與程式骨架

```powershell
dotnet new xunit -n ProductHub.Api.Tests
dotnet add ProductHub.Api.Tests reference ProductHub.Api
dotnet test
```

```csharp
[Fact]
public async Task CreateProductAsync_Should_Create_Product()
{
    // Arrange
    var request = new CreateProductRequest("Keyboard", 1200m, 10);

    // Act
    var product = await service.CreateProductAsync(request);

    // Assert
    Assert.Equal("Keyboard", product.Name);
}
```

### 常見踩雷 / 排查

- 測試依賴真實資料庫：容易不穩。
- 只測 happy path：要補錯誤情境。
- 測 Controller 太細：商業邏輯優先測 Service。

### 作業

補三個測試：建立成功、找不到商品、價格不合法。

### Junior 提醒

測試不是為了覆蓋率數字，而是為了保護重要行為。

### 一句話收斂

今天開始替後端功能買保險。

---

## Day 26：前端測試

### 今天要學什麼

用 Vitest 測試 Pinia store 與 component 的基本行為。

### 為什麼重要

前端狀態與互動越多，越需要測試保護 loading、error、empty 這些容易被忽略的情境。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Vitest | Vite 生態測試工具 |
| Mock | 假資料或假 service |
| Component Test | 測元件渲染與互動 |

### 實作步驟與程式骨架

```ts
import { describe, expect, it } from 'vitest'

describe('product store', () => {
  it('loads products', async () => {
    // arrange fake api
    // act loadProducts
    // assert products length
    expect(true).toBe(true)
  })
})
```

### 常見踩雷 / 排查

- 測試真的打 API：慢且不穩。
- 只測畫面文字：可以補互動行為。
- 沒重設 store：測試互相污染。

### 作業

測試商品清單在 empty state 顯示「目前沒有商品」。

### Junior 提醒

前端測試的價值在於保護使用者看得到、點得到的行為。

### 一句話收斂

今天讓前端互動不只靠手動點。

---

## Day 27：Build 與效能檢查

### 今天要學什麼

後端用 release build，前端用 `npm run build`，檢查 bundle、API response 與基本效能。

### 為什麼重要

開發環境能跑不等於正式環境能跑。Build 階段會抓出型別、路徑、環境變數問題。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Release Build | 正式編譯 |
| Bundle | 前端打包輸出 |
| Source Map | 除錯用對應檔 |
| Lazy Loading | 延遲載入頁面 |

### 實作步驟與程式骨架

```powershell
cd ProductHub/backend/ProductHub.Api
dotnet build -c Release

cd ProductHub/frontend/product-hub-web
npm run build
```

### 常見踩雷 / 排查

- dev 可跑，build 失敗：通常是 TypeScript 或 import path 問題。
- 前端正式 API URL 錯：檢查 `.env.production`。
- bundle 過大：route-level lazy loading、移除不必要套件。

### 作業

建立 `docs/build-checklist.md`，記錄 build 指令與常見錯誤。

### Junior 提醒

每次交付前至少跑一次後端 release build 與前端 production build。

### 一句話收斂

今天把開發中的作品推向可交付狀態。

---

## Day 28：部署

### 今天要學什麼

理解 API 與 Vue SPA 的部署方式：分開部署、環境設定、HTTPS、資料庫連線。

### 為什麼重要

部署不是最後一個按鈕，而是系統真正面對使用者的開始。

### 關鍵名詞對照

| 名詞 | 說明 |
| --- | --- |
| Hosting | 放置後端或前端的環境 |
| Environment Variable | 正式環境設定 |
| HTTPS | 加密連線 |
| Migration | 部署時更新資料庫結構 |

### 實作步驟與程式骨架

部署檢查：

```text
1. API 有正式 connection string
2. API 有正式 JWT secret
3. CORS 允許正式前端 domain
4. Vue `.env.production` 指向正式 API
5. 資料庫 migration 已執行
6. HTTPS 可正常連線
7. 登入、清單、新增、編輯、刪除流程測過
```

### 常見踩雷 / 排查

- 正式環境 CORS 忘記改 domain。
- 前端 build 時吃到錯的 API URL。
- connection string 放在 repo。
- SPA 重新整理 404：靜態網站要設定 fallback 到 `index.html`。

### 作業

寫一份 `docs/deploy.md`，記錄你的部署目標與環境設定。

### Junior 提醒

部署最怕「我電腦可以」。文件與環境變數要寫清楚。

### 一句話收斂

今天讓專案離開本機。

---

## Day 29：README 與交付文件

### 今天要學什麼

整理 README、API 文件、啟動步驟、測試帳號與功能截圖。

### 為什麼重要

好專案不只 code 能跑，也要讓別人能理解、啟動、測試與維護。

### 關鍵名詞對照

| 文件 | 內容 |
| --- | --- |
| README | 專案介紹與啟動方式 |
| API Docs | endpoint 與 request/response |
| ERD | 資料表關係 |
| Checklist | 測試與交付項目 |

### 實作步驟與程式骨架

```text
# ProductHub

## 功能
- 商品 CRUD
- 搜尋、排序、分頁
- 登入與角色授權

## 技術
- ASP.NET Core Web API
- EF Core
- Vue 3 + Vite + TypeScript
- Pinia

## 本機啟動
1. 啟動 API
2. 執行 migration
3. 啟動 Vue dev server

## 測試帳號
- Admin: admin@example.com
```

### 常見踩雷 / 排查

- README 只有專案名稱：別人無法啟動。
- 忘記寫環境變數：部署會卡住。
- 沒有測試帳號：demo 會浪費時間。

### 作業

完成 README，讓另一個人照著文件可以啟動專案。

### Junior 提醒

文件不是額外工作，它是降低交接成本的工程資產。

### 一句話收斂

今天把作品整理成別人看得懂的專案。

---

## Day 30：全端總複習與作品檢查

### 今天要學什麼

回顧 30 天完成的全端流程，使用 checklist 檢查專案品質。

### 為什麼重要

學完 API 與 Vue 不代表能做專案。能把功能、錯誤、權限、部署與文件串起來，才是全端能力的開始。

### 關鍵名詞對照

| 能力 | 你現在應該能做到 |
| --- | --- |
| API 設計 | 能設計 CRUD endpoint 與 DTO |
| 前端整合 | 能用 Vue 呼叫 API 並處理狀態 |
| 資料庫 | 能用 EF Core migration 管理 schema |
| 權限 | 能做基本登入與 API 保護 |
| 部署 | 能說明前後端如何上線 |

### 實作步驟與程式骨架

最終檢查：

```text
後端
- dotnet build -c Release 通過
- dotnet test 通過
- Swagger 可測核心 API
- CORS 沒有開過大
- secret 沒有 commit

前端
- npm run build 通過
- 登入流程正常
- CRUD 流程正常
- loading / empty / error 都有畫面
- API URL 由環境變數控制

文件
- README 可照著啟動
- deploy.md 寫清楚正式環境設定
- 測試帳號與 demo flow 清楚
```

### 常見踩雷 / 排查

- 只 demo happy path：也要測錯誤、空資料、無權限。
- 忘記清除測試資料：正式 demo 容易混亂。
- 沒有 commit 節奏：作品看不出開發過程。

### 作業

錄一段 3 到 5 分鐘 demo：登入、商品清單、搜尋、新增、編輯、刪除、登出。

### Junior 提醒

作品不是功能越多越好，而是核心流程穩、錯誤處理清楚、文件完整。

### 一句話收斂

30 天的終點不是結束，而是你已經有能力獨立做一個小型全端系統。

---

## 常見全端 Debug 流程

當畫面壞掉時，不要直接猜。照這條路線查：

```text
1. 瀏覽器畫面有沒有錯誤訊息
2. DevTools Console 有沒有 JS error
3. DevTools Network request 是否送出
4. request URL / method / payload 是否正確
5. API response status code 是多少
6. 後端 terminal / log 有沒有 exception
7. DB 是否真的有資料
8. 前端 store state 是否更新
```

| 現象 | 優先檢查 |
| --- | --- |
| 畫面空白 | Console、route、component import |
| 一直 loading | request 是否結束、finally 是否執行 |
| CORS error | 後端 CORS policy、前端 origin |
| 404 | route、API URL、controller route |
| 400 | request body、DTO validation |
| 401 | token 是否存在與過期 |
| 403 | role / policy 是否符合 |
| 500 | 後端 log、exception、connection string |

## 下一步學習建議

- 把簡化登入換成 ASP.NET Core Identity 或外部 OAuth/OIDC。
- 將圖片上傳改成雲端 Object Storage。
- 為 API 加上 OpenAPI contract 與自動產生 TypeScript client。
- 加入 CI/CD，自動跑 `dotnet test`、`npm run build`。
- 將部署拆成 dev / staging / production。
