# C# ASP.NET MVC 實作：30 天打造網站應用程式 2026 更新整理

## 來源

- 原系列：<https://ithelp.ithome.com.tw/users/20150166/ironman/4959>
- RSS：<https://ithelp.ithome.com.tw/rss/series/4959>
- 作者：Coding小僧
- 原始系列年份：2022 iThome 鐵人賽
- 2026 更新參考：
  - .NET 版本支援：<https://learn.microsoft.com/en-us/dotnet/core/releases-and-support>
  - ASP.NET Core MVC 概觀：<https://learn.microsoft.com/en-us/aspnet/core/mvc/overview>
  - ASP.NET Core MVC Views：<https://learn.microsoft.com/en-us/aspnet/core/mvc/views/>

## 來源提醒與 2026 更新方向

原系列主要使用 `ASP.NET MVC 5 / .NET Framework`、Visual Studio、Entity Framework 6、Azure App Service 與 SQL Database，目標是讓初學者做出一個小型購物網站。這份筆記會保留原系列的學習順序，但用 2026 年的角度補上判斷：

- 新專案建議優先使用 `ASP.NET Core MVC`，目標框架可選 `.NET 10 LTS`；若公司既有系統仍是 MVC5 / .NET Framework，才以維護與遷移角度學 MVC5。
- 原系列的 `Controller / View / Model / Razor / EF / Routing / Validation / Partial View` 概念仍然有價值，因為 ASP.NET Core MVC 也延續 MVC 的核心分工。
- 原文中的 `ViewBag`、`ViewData`、`TempData` 可作為理解資料傳遞工具，但新專案應優先使用強型別 ViewModel。
- 原文用 `PagedList.Mvc`、`Newtonsoft.Json`、連線字串加密等作法，現在要依專案版本調整；ASP.NET Core 常見替代是 `X.PagedList.Mvc.Core`、內建 `System.Text.Json`、User Secrets / Key Vault / 環境變數。

## 這份筆記怎麼讀

如果你是 junior developer，建議把這 30 天拆成四段：

| 階段 | Day | 學習主軸 | 2026 建議 |
| --- | --- | --- | --- |
| MVC 基礎 | 1-9 | MVC、Controller、View、表單、驗證 | 用 ASP.NET Core MVC 重做一次最小範例 |
| 資料庫與 CRUD | 10-12 | EF、CRUD、授權過濾器 | 用 EF Core、Identity 或授權 policy 補強 |
| 購物網站實作 | 13-20 | 商品、會員、購物車、訂單 | 補上 Service 層、ViewModel、測試 |
| 部署與補充 | 21-30 | Azure、連線字串、Partial View、Route、JSON、分頁 | 改用雲端設定、CI/CD、現代套件 |

## 整體心智模型

把 MVC 想成餐廳：

- `Controller` 是櫃台，接收客人的 request，判斷要做什麼。
- `Model` 是廚房與資料，負責商業規則、資料狀態與資料庫互動。
- `View` 是上菜畫面，負責把結果呈現給使用者。

實務上更精確地說，MVC 是一種關注點分離方式。Controller 不應該塞滿商業邏輯，View 不應該查資料庫，Model / Service / Repository 應該承擔可測試、可重用的規則與資料處理。

## 30 天來源清單

| Day | 主題 | 原文 |
| --- | --- | --- |
| 1 | 前言、ASP.NET 與 MVC | <https://ithelp.ithome.com.tw/articles/10287423> |
| 2 | 建立第一個網站應用程式 | <https://ithelp.ithome.com.tw/articles/10291477> |
| 3 | 新增 Controller | <https://ithelp.ithome.com.tw/articles/10292310> |
| 4 | 新增 View 與 ViewData / ViewBag / TempData | <https://ithelp.ithome.com.tw/articles/10293122> |
| 5 | 物件 Model 資料傳遞 | <https://ithelp.ithome.com.tw/articles/10293839> |
| 6 | 表單 form 資料傳遞 | <https://ithelp.ithome.com.tw/articles/10294588> |
| 7 | Bootstrap 套件 | <https://ithelp.ithome.com.tw/articles/10295312> |
| 8 | HTML Helper | <https://ithelp.ithome.com.tw/articles/10296011> |
| 9 | 資料驗證 | <https://ithelp.ithome.com.tw/articles/10296740> |
| 10 | Entity Framework 存取資料庫 | <https://ithelp.ithome.com.tw/articles/10297273> |
| 11 | 簡易 CRUD | <https://ithelp.ithome.com.tw/articles/10297379> |
| 12 | 授權過濾器驗證 | <https://ithelp.ithome.com.tw/articles/10298529> |
| 13 | 購物中心實作一 | <https://ithelp.ithome.com.tw/articles/10299387> |
| 14 | 購物中心實作二 | <https://ithelp.ithome.com.tw/articles/10300136> |
| 15 | 購物中心實作三 | <https://ithelp.ithome.com.tw/articles/10300727> |
| 16 | 購物中心實作四 | <https://ithelp.ithome.com.tw/articles/10301398> |
| 17 | 購物中心實作五 | <https://ithelp.ithome.com.tw/articles/10301930> |
| 18 | 購物中心實作六 | <https://ithelp.ithome.com.tw/articles/10302569> |
| 19 | 購物中心實作七 | <https://ithelp.ithome.com.tw/articles/10303196> |
| 20 | 購物中心實作八 | <https://ithelp.ithome.com.tw/articles/10303770> |
| 21 | Azure 帳號建立 | <https://ithelp.ithome.com.tw/articles/10304325> |
| 22 | Azure 資料庫使用 | <https://ithelp.ithome.com.tw/articles/10304834> |
| 23 | EF 連線字串加密 | <https://ithelp.ithome.com.tw/articles/10305368> |
| 24 | Partial View | <https://ithelp.ithome.com.tw/articles/10305820> |
| 25 | Route | <https://ithelp.ithome.com.tw/articles/10306284> |
| 26 | 各種 ActionResult | <https://ithelp.ithome.com.tw/articles/10306746> |
| 27 | 自訂 Helper | <https://ithelp.ithome.com.tw/articles/10307260> |
| 28 | JSON 資料格式 | <https://ithelp.ithome.com.tw/articles/10307708> |
| 29 | PagedList 套件 | <https://ithelp.ithome.com.tw/articles/10308115> |
| 30 | 結語 | <https://ithelp.ithome.com.tw/articles/10308421> |

---

## Day 1：前言、ASP.NET 與 MVC

### 這篇文章主要在講什麼

原文先建立 ASP.NET、ASP.NET Core、MVC 的基本認識，並說明系列目標是用 C# 與 MVC 做出小型購物網站。重點不是背框架歷史，而是知道 MVC 為什麼能讓網頁程式比較容易維護。

### 為什麼需要這個概念

如果所有 UI、資料庫查詢、商業規則都塞在同一個檔案，需求一變就很容易牽一髮動全身。MVC 把責任拆開，讓你能分別修改畫面、流程與資料規則。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Model | Model | 商業邏輯、資料狀態、資料庫資料 |
| View | View | HTML / CSS / JavaScript 畫面呈現 |
| Controller | Controller | 接收 request、呼叫 Model、選擇 View |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立一個 ASP.NET Core MVC 專案。
2. 確認 `Controllers`、`Models`、`Views` 三個資料夾。
3. 讓 Controller 回傳 View。
4. 在 View 顯示 Model 資料。
5. 用瀏覽器確認 URL 對應到正確頁面。

```csharp
// 範例用途：用現代 ASP.NET Core MVC 展示 MVC 三者如何合作。
// 參數說明：Index 沒有輸入參數，代表首頁只顯示固定資料。
// 回傳結果 / 副作用：回傳 ViewResult，Razor View 會被渲染成 HTML。
public sealed record ProductSummaryViewModel(string Name, decimal Price);

public class HomeController : Controller
{
    public IActionResult Index()
    {
        // Model / ViewModel：準備畫面需要的資料。
        var model = new ProductSummaryViewModel("鍵盤", 1200m);

        // Controller：選擇 View，並把 Model 傳給 View。
        return View(model);
    }
}
```

View 範例：

```cshtml
@model ProductSummaryViewModel

<!-- View：只負責呈現資料，不在這裡查資料庫。 -->
<h1>@Model.Name</h1>
<p>價格：@Model.Price</p>
```

注意事項：
- 不要把資料庫查詢寫在 View，會讓畫面和資料存取綁死。
- Controller 可以協調流程，但不要變成所有商業規則的垃圾桶。
- 新專案優先選 ASP.NET Core MVC；MVC5 適合維護舊系統時學。

### 負面例子 / 錯誤用法

錯誤做法：在 Razor View 直接 new DbContext 查商品。

問題：畫面測試困難、資料庫連線散落各處、權限與交易不容易管理。

修正方向：Controller 呼叫 Service，Service 查資料後轉成 ViewModel 給 View。

### 小練習

建立 `ProductController`，讓 `/Product/Detail` 顯示一筆商品名稱與價格。

### 一句話總結

MVC 的核心是把「收到請求、處理資料、呈現畫面」分工，讓程式更好改、更好測。

---

## Day 2：建立第一個網站應用程式

### 這篇文章主要在講什麼

原文示範安裝 Visual Studio、建立 ASP.NET MVC 專案、執行預設首頁、修改 `Index.cshtml` 與 Layout。

### 學完這篇你應該會做到什麼

你應該能建立一個 MVC 專案、啟動本機網站、修改首頁內容，並知道 Layout 是所有頁面的共同外框。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| IDE | Visual Studio | 建立、編輯、執行專案 |
| View | Index.cshtml | 首頁主要內容 |
| Layout | _Layout.cshtml / Layout.cshtml | 共用導覽列、頁尾與外框 |
| RenderBody | RenderBody | 把每個 View 的內容放進 Layout |

### 完整實作流程、範例與注意事項

完整流程：

1. 安裝 Visual Studio 2022 或更新版本，勾選 ASP.NET 與網頁開發工作負載。
2. 建立 ASP.NET Core Web App，選 MVC。
3. 執行專案，確認首頁可開啟。
4. 修改 `Views/Home/Index.cshtml`。
5. 修改 `Views/Shared/_Layout.cshtml` 的標題、導覽列或頁尾。

```cshtml
@{
    ViewData["Title"] = "我的第一個 MVC 首頁";
}

<h1>我的第一個 ASP.NET Core MVC 程式</h1>
<p>這段內容來自 Views/Home/Index.cshtml。</p>
```

注意事項：
- `ViewData["Title"]` 常被 Layout 讀取，用於瀏覽器標題。
- Layout 適合放共用元素，不適合塞單一頁面的特殊商業邏輯。
- 如果改 View 後沒變，先確認瀏覽器有重新整理、專案有成功建置。

### 負面例子 / 錯誤用法

錯誤做法：每個 View 都複製一份完整導覽列 HTML。

問題：日後導覽列要加一個選單，所有 View 都要改。

修正方向：把共用外框放在 `_Layout.cshtml`，頁面差異留在各自 View。

### 小練習

新增 `About.cshtml`，並在導覽列加上「關於本站」連結。

### 一句話總結

第一個 MVC 專案的重點，是看懂 View 與 Layout 如何組成最後的 HTML 頁面。

---

## Day 3：新增 Controller

### 這篇文章主要在講什麼

原文介紹 Controller 與 Action Method，並示範用 GET URL 參數回傳文字或 View。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Controller | DemoController | 接收路由請求 |
| Action | Index、ShowHelloWorld、ShowPrice | 對應 URL 的方法 |
| ActionResult | ActionResult / ViewResult | 表示 action 執行後要回傳的結果 |

### 完整實作流程、範例與注意事項

完整流程：

1. 新增 `ProductsController`。
2. 新增 `Detail(int id)` action。
3. 從 URL 取得商品 id。
4. 回傳文字或 View。
5. 用 `/Products/Detail/10` 測試。

```csharp
public class ProductsController : Controller
{
    // Controller：ProductsController 負責商品相關 request。
    // 參數說明：id 來自 URL，例如 /Products/Detail/10。
    // 回傳結果：Content 會直接回傳文字給瀏覽器。
    [HttpGet]
    public IActionResult Detail(int id)
    {
        return Content($"商品編號：{id}");
    }
}
```

注意事項：
- GET 適合查詢，不適合修改資料。
- URL 參數是外部輸入，不能假設一定正確。
- Action 名稱與路由規則會影響使用者看到的 URL。

### 負面例子 / 錯誤用法

錯誤做法：用 GET `/Products/Delete/10` 直接刪除商品。

問題：使用者或搜尋引擎誤觸連結就可能改變資料。

修正方向：查詢用 GET，新增、修改、刪除用 POST / PUT / DELETE，並加上 CSRF 保護。

### 小練習

建立 `/Products/Search?keyword=keyboard`，回傳搜尋關鍵字。

### 一句話總結

Controller 是 MVC 的入口，Action Method 是 URL request 實際會呼叫的方法。

---

## Day 4：新增 View 與 ViewData / ViewBag / TempData

### 這篇文章主要在講什麼

原文說明 Controller 如何把資料傳給 View，包含 `ViewData`、`ViewBag`、`TempData`。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| ViewData | ViewData | Dictionary 型弱型別資料 |
| ViewBag | ViewBag | dynamic 型弱型別資料 |
| TempData | TempData | 可跨一次 redirect 的暫存資料 |
| View | .cshtml | 顯示 Controller 傳來的資料 |

### 完整實作流程、範例與注意事項

完整流程：

1. Controller 準備要顯示的訊息。
2. 使用 `ViewData` 或 `ViewBag` 傳入 View。
3. 在 `.cshtml` 顯示內容。
4. 使用 `TempData` 顯示一次性成功訊息。
5. 改成 ViewModel 比較可維護。

```csharp
public IActionResult Saved()
{
    // TempData：適合 redirect 後顯示一次性訊息。
    TempData["Message"] = "商品已儲存";
    return RedirectToAction("Index");
}

public IActionResult Index()
{
    // ViewBag：快速展示可以用，但大型頁面建議改 ViewModel。
    ViewBag.PageTitle = "商品列表";
    return View();
}
```

注意事項：
- `ViewBag.ProductNmae` 拼錯時編譯不會提醒，容易到執行期才爆。
- `TempData` 通常只讀一次，適合 flash message。
- 新專案優先用 ViewModel，把頁面需要的資料定義清楚。

### 負面例子 / 錯誤用法

錯誤做法：整個頁面都靠十幾個 `ViewBag` 欄位組合。

問題：欄位名稱沒有型別檢查，維護者很難知道 View 需要什麼資料。

修正方向：建立 `ProductListViewModel`。

### 小練習

把 `ViewBag.PageTitle` 改成強型別 `PageTitle` 屬性。

### 一句話總結

弱型別資料傳遞適合小訊息，正式頁面應該用 ViewModel 讓資料契約明確。

---

## Day 5：物件 Model 資料傳遞

### 這篇文章主要在講什麼

原文從單一物件與 List 角度，說明如何把 Model 傳給 View 顯示。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Model | Member / Product | 表示資料物件 |
| Strongly Typed View | `@model` | 讓 View 明確知道資料型別 |
| List Model | `List<T>` | 傳多筆資料給 View |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立 `ProductListItemViewModel`。
2. Controller 建立商品清單。
3. `return View(products)`。
4. View 用 `@model IReadOnlyList<ProductListItemViewModel>`。
5. 用 `foreach` 顯示每筆商品。

```csharp
public sealed record ProductListItemViewModel(int Id, string Name, decimal Price);

public IActionResult Index()
{
    // 範例用途：準備商品列表頁需要的資料。
    // 回傳結果：View 會取得強型別商品清單。
    var products = new List<ProductListItemViewModel>
    {
        new(1, "滑鼠", 590m),
        new(2, "鍵盤", 1200m)
    };

    return View(products);
}
```

```cshtml
@model IReadOnlyList<ProductListItemViewModel>

@foreach (var product in Model)
{
    <p>@product.Name：@product.Price</p>
}
```

注意事項：
- ViewModel 不一定等於資料庫 Entity。
- 顯示頁面通常只需要部分欄位，不要把整個 Entity 原封不動丟給 View。
- 使用 `IReadOnlyList<T>` 可表達 View 只讀取、不修改集合。

### 負面例子 / 錯誤用法

錯誤做法：把 `ProductEntity` 直接傳給 View，裡面含成本、供應商內部欄位。

問題：容易誤顯示敏感資料，也讓 View 綁死資料庫結構。

修正方向：建立專門給畫面的 ViewModel。

### 小練習

替商品清單加上 `IsOnSale`，並在 View 顯示「特價中」。

### 一句話總結

強型別 ViewModel 讓 Controller 與 View 的資料契約變清楚，是 MVC 維護性的基礎。

---

## Day 6：表單 form 資料傳遞

### 這篇文章主要在講什麼

原文示範 HTML form 如何把使用者輸入送回 Controller，並說明 GET / POST 差異。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Form | form | 收集使用者輸入 |
| GET | HttpGet | 查詢資料，參數通常在 URL |
| POST | HttpPost | 送出資料，通常會新增或修改狀態 |
| Model Binding | 參數繫結 | 把 request 資料轉成 C# 物件 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立 `CreateProductRequest`。
2. GET action 顯示表單。
3. POST action 接收表單。
4. 檢查 `ModelState.IsValid`。
5. 成功後 redirect，避免重新整理重複送出。

```csharp
public sealed class CreateProductRequest
{
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
}

[HttpGet]
public IActionResult Create() => View(new CreateProductRequest());

[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(CreateProductRequest request)
{
    // 參數說明：request 由表單欄位 Name、Price model binding 而來。
    // 回傳結果 / 副作用：真實專案會新增商品；此處用 redirect 模擬成功流程。
    if (!ModelState.IsValid)
    {
        return View(request);
    }

    TempData["Message"] = $"已新增商品：{request.Name}";
    return RedirectToAction("Index");
}
```

注意事項：
- 修改資料的表單要用 POST。
- POST 表單要加 anti-forgery token。
- 成功後用 Post/Redirect/Get，避免 F5 重複送出。

### 負面例子 / 錯誤用法

錯誤做法：POST 成功後直接 `return View()`。

問題：使用者重新整理頁面時，瀏覽器可能再次提交表單。

修正方向：成功後 `RedirectToAction`。

### 小練習

替 `CreateProductRequest` 增加 `Stock` 欄位並在表單中接收。

### 一句話總結

表單處理的基本安全線是 POST、驗證、anti-forgery、成功後 redirect。

---

## Day 7：Bootstrap 套件

### 這篇文章主要在講什麼

原文介紹 Bootstrap 如何快速套用版面、按鈕、表格與 responsive 樣式。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| CSS Framework | Bootstrap | 提供現成排版與元件樣式 |
| Grid | container / row / col | 建立響應式版面 |
| Component | btn / table / navbar | 常用 UI 元件 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認專案已載入 Bootstrap CSS。
2. 用 `container` 包住主要內容。
3. 用 `table table-striped` 顯示清單。
4. 用 `btn btn-primary` 表示主要操作。
5. 用手機寬度檢查排版是否可讀。

```cshtml
<div class="container">
    <a class="btn btn-primary" asp-controller="Products" asp-action="Create">新增商品</a>

    <table class="table table-striped mt-3">
        <thead>
            <tr><th>商品</th><th>價格</th></tr>
        </thead>
        <tbody>
            <tr><td>鍵盤</td><td>1200</td></tr>
        </tbody>
    </table>
</div>
```

注意事項：
- Bootstrap 能加快開發，但不能取代資訊架構與可用性設計。
- 不要只靠顏色表達狀態，仍要有文字或圖示。
- 套件版本不同時 class 或元件寫法可能會變。

### 負面例子 / 錯誤用法

錯誤做法：頁面每個按鈕都用 `btn-primary`。

問題：使用者分不出主要動作與次要動作。

修正方向：主要動作用 primary，取消或返回用 secondary / link 樣式。

### 小練習

把商品清單改成 responsive table，並替價格欄靠右對齊。

### 一句話總結

Bootstrap 是快速建立一致 UI 的工具，但真正的重點仍是清楚的版面與操作層級。

---

## Day 8：HTML Helper

### 這篇文章主要在講什麼

原文介紹 MVC5 的 HTML Helper，例如產生 input、label、validation message。ASP.NET Core MVC 則常用 Tag Helper。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| HTML Helper | `Html.TextBoxFor` | 用 C# helper 產生 HTML |
| Tag Helper | `asp-for` | ASP.NET Core 推薦的 Razor 標籤輔助 |
| Model Expression | `m => m.Name` | 綁定模型欄位 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立 ViewModel。
2. 在 Razor 宣告 `@model`。
3. 用 `asp-for` 產生 label 與 input。
4. 用 validation span 顯示錯誤。
5. 送出後確認欄位 name 與 model binding 對應。

```cshtml
@model CreateProductRequest

<form asp-action="Create" method="post">
    @Html.AntiForgeryToken()

    <label asp-for="Name"></label>
    <input asp-for="Name" class="form-control" />
    <span asp-validation-for="Name" class="text-danger"></span>

    <button class="btn btn-primary" type="submit">儲存</button>
</form>
```

注意事項：
- MVC5 常見 `Html.BeginForm`、`Html.EditorFor`；ASP.NET Core 可用 Tag Helper 讓 HTML 更直覺。
- `asp-for` 會依 Model metadata 產生 name / id。
- 不要手刻錯誤的 name，否則 model binding 會收不到。

### 負面例子 / 錯誤用法

錯誤做法：`<input name="product_name">`，但 ViewModel 屬性叫 `Name`。

問題：POST 後 `Name` 綁不到值。

修正方向：用 `asp-for="Name"` 或確保 name 與屬性一致。

### 小練習

替 `Price` 欄位加上 label、input 與 validation message。

### 一句話總結

Helper 的價值是讓表單欄位與 Model 綁定一致，減少手刻 HTML 的錯誤。

---

## Day 9：資料驗證

### 這篇文章主要在講什麼

原文介紹 Data Annotations，例如 Required、StringLength、Range，並搭配 ModelState 驗證。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Validation Attribute | Required / Range | 宣告欄位限制 |
| ModelState | ModelState.IsValid | 判斷 request 是否通過驗證 |
| Validation Message | ValidationMessageFor / asp-validation-for | 在畫面顯示錯誤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 在 request model 加 Data Annotations。
2. POST action 檢查 `ModelState.IsValid`。
3. 失敗時回到原 View，保留使用者輸入。
4. View 顯示欄位錯誤。
5. 成功才呼叫 service 寫入資料。

```csharp
public sealed class CreateProductRequest
{
    [Required(ErrorMessage = "商品名稱必填")]
    [StringLength(50, ErrorMessage = "商品名稱最多 50 字")]
    public string Name { get; set; } = "";

    [Range(1, 999999, ErrorMessage = "價格必須大於 0")]
    public decimal Price { get; set; }
}
```

注意事項：
- 前端驗證是使用者體驗，後端驗證才是安全底線。
- 驗證規則不要只放 JavaScript。
- 複雜商業規則可放到 Service 或自訂 validator。

### 負面例子 / 錯誤用法

錯誤做法：Controller 不檢查 `ModelState.IsValid` 就直接寫入資料庫。

問題：空商品名稱、負價格都可能進資料庫。

修正方向：POST action 第一段先驗證，失敗就回 View。

### 小練習

加入 `Stock` 欄位，限制庫存必須在 0 到 9999。

### 一句話總結

驗證不是裝飾表單，而是保護資料品質與系統規則的第一道門。

---

## Day 10：Entity Framework 存取與操作資料庫

### 這篇文章主要在講什麼

原文介紹用 Entity Framework 連接資料庫、建立 Entity、透過 DbContext 查詢資料。2026 新專案建議使用 EF Core。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| ORM | Entity Framework | 用 C# 物件操作資料庫 |
| DbContext | DbContext | 資料庫工作階段與查詢入口 |
| DbSet | DbSet<Product> | 對應資料表 |
| Entity | Product | 對應資料列 |

### 完整實作流程、範例與注意事項

完整流程：

1. 安裝 EF Core provider，例如 SQL Server provider。
2. 建立 Entity。
3. 建立 `AppDbContext`。
4. 在 DI 註冊 DbContext 與連線字串。
5. 使用 migration 建立資料庫。

```csharp
public sealed class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
}

public sealed class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    // DbSet：Product Entity 對應 Products 資料表。
    public DbSet<Product> Products => Set<Product>();
}
```

注意事項：
- Entity 是資料庫模型，不一定適合直接當 ViewModel。
- 查詢大量資料時不要先 `ToList()` 再篩選。
- migration 要納入版本控管，讓資料庫結構可追蹤。

### 負面例子 / 錯誤用法

錯誤做法：每個 action 都手動 new DbContext，且忘記釋放。

問題：生命週期混亂、測試困難、連線管理不穩。

修正方向：使用 DI 注入 DbContext 或 Service。

### 小練習

建立 `Category` Entity，讓 `Product` 擁有 `CategoryId`。

### 一句話總結

EF 的價值是用物件模型操作資料庫，但仍要理解查詢、連線與資料模型邊界。

---

## Day 11：簡易 CRUD

### 這篇文章主要在講什麼

原文示範 Create、Read、Update、Delete 四種基本資料操作，是管理後台與資料維護功能的核心。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Create | 新增 | 建立資料 |
| Read | 查詢 / Details / Index | 顯示資料 |
| Update | Edit | 修改資料 |
| Delete | Delete | 刪除資料 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立列表頁 `Index`。
2. 建立新增頁 `Create`。
3. 建立編輯頁 `Edit`。
4. 建立刪除確認頁 `Delete`.
5. 每個 POST 都驗證輸入與資料是否存在。

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Edit(int id, EditProductRequest request)
{
    // 參數說明：id 來自路由，request 來自表單。
    // 副作用：找到商品後更新資料庫。
    if (!ModelState.IsValid) return View(request);

    var product = await _db.Products.FindAsync(id);
    if (product is null) return NotFound();

    product.Name = request.Name;
    product.Price = request.Price;
    await _db.SaveChangesAsync();

    return RedirectToAction(nameof(Index));
}
```

注意事項：
- 編輯時要確認 route id 與資料存在。
- 刪除要用 POST，不要用 GET 直接刪。
- 大型專案建議把 CRUD 邏輯移到 Service。

### 負面例子 / 錯誤用法

錯誤做法：使用者送什麼欄位就整個 Entity update。

問題：可能產生 over-posting，讓不該被修改的欄位被改掉。

修正方向：使用 request DTO，只更新允許修改的欄位。

### 小練習

替 Delete 加上確認頁，POST 後才真的刪除。

### 一句話總結

CRUD 看似簡單，但真正重要的是驗證、資料存在檢查與避免不該改的欄位被改。

---

## Day 12：授權過濾器驗證

### 這篇文章主要在講什麼

原文介紹授權過濾器，讓特定 Controller 或 Action 只有登入或符合權限的人能使用。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Authorization Filter | 授權過濾器 | 在 action 前檢查是否可進入 |
| Authorize | AuthorizeAttribute | 限制登入或角色 |
| Anonymous | AllowAnonymous | 允許未登入存取 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認系統有身份驗證機制。
2. 在後台 Controller 加上 `[Authorize]`。
3. 對公開頁面使用 `[AllowAnonymous]`。
4. 需要角色時加 roles 或 policy。
5. 測試未登入、一般會員、管理員三種情境。

```csharp
[Authorize(Roles = "Admin")]
public class AdminProductsController : Controller
{
    // 只有 Admin 可以進入商品管理頁。
    public IActionResult Index() => View();
}
```

注意事項：
- 驗證 authentication 是「你是誰」，授權 authorization 是「你能做什麼」。
- 不能只靠前端隱藏按鈕，後端 action 一定要檢查。
- 角色過多時可考慮 policy-based authorization。

### 負面例子 / 錯誤用法

錯誤做法：只有 View 判斷不是管理員就不顯示刪除按鈕。

問題：使用者仍可直接打刪除 URL 或送 request。

修正方向：Controller / API 層必須加授權檢查。

### 小練習

建立 `Member` 與 `Admin` 兩種角色，限制後台只能 Admin 進入。

### 一句話總結

權限檢查必須放在伺服器端，UI 隱藏只是體驗，不是安全。

---

## Day 13：購物中心實作一

### 這篇文章主要在講什麼

原系列從這裡開始進入購物網站實作，通常會先建立專案結構、資料模型與基礎頁面。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Product | 商品 | 可被瀏覽與加入購物車的資料 |
| Category | 分類 | 商品分群 |
| Storefront | 購物中心首頁 | 顯示商品與入口 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立購物網站專案。
2. 定義 Product / Category。
3. 建立商品列表頁。
4. 建立商品詳細頁。
5. 用假資料或 seed data 驗證畫面。

```csharp
public sealed record ProductCardViewModel(
    int Id,
    string Name,
    decimal Price,
    string CategoryName);
```

注意事項：
- 一開始先做最小可跑版本，不要急著做完整電商。
- 商品列表頁只放卡片需要的欄位。
- 分類與商品是常見一對多關係。

### 負面例子 / 錯誤用法

錯誤做法：第一天就設計優惠券、金流、物流、會員等完整資料表。

問題：過早複雜化，容易卡在還沒驗證的需求。

修正方向：先完成商品瀏覽與詳細頁，再逐步加購物車與訂單。

### 小練習

建立三筆商品 seed data，讓首頁可以顯示商品卡片。

### 一句話總結

購物網站實作要先讓商品能被瀏覽，這是後續購物車與訂單的基礎。

---

## Day 14：購物中心實作二

### 這篇文章主要在講什麼

這一段通常會延伸商品資料維護、分類或列表呈現，讓網站從靜態頁面往資料驅動畫面前進。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Product List | 商品列表 | 顯示多筆商品 |
| Product Detail | 商品詳細 | 顯示單筆商品 |
| Query | 查詢 | 從資料來源取得符合條件的商品 |

### 完整實作流程、範例與注意事項

完整流程：

1. 在 Controller 接收分類或關鍵字。
2. Service 查詢商品。
3. 轉成 ViewModel。
4. View 顯示列表。
5. 測試無資料時的畫面。

```csharp
public async Task<IActionResult> Index(string? keyword)
{
    // 參數說明：keyword 來自查詢字串，用於商品名稱搜尋。
    var query = _db.Products.AsQueryable();

    if (!string.IsNullOrWhiteSpace(keyword))
    {
        query = query.Where(p => p.Name.Contains(keyword));
    }

    var model = await query
        .Select(p => new ProductCardViewModel(p.Id, p.Name, p.Price, p.Category.Name))
        .ToListAsync();

    return View(model);
}
```

注意事項：
- 搜尋條件要處理 null / 空白。
- 大量資料應加分頁，不要一次載入全部。
- 顯示資料時要處理無結果狀態。

### 負面例子 / 錯誤用法

錯誤做法：`_db.Products.ToList().Where(...)`。

問題：先把全部資料拉到記憶體，資料大時效能差。

修正方向：保持 `IQueryable`，在資料庫端完成篩選。

### 小練習

加入分類篩選 `categoryId`。

### 一句話總結

列表頁的關鍵是查詢條件、資料投影與空結果處理。

---

## Day 15：購物中心實作三

### 這篇文章主要在講什麼

購物網站開始需要會員或登入狀態，才能做個人化購物車、訂單與後台操作。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Member | 會員 | 系統使用者 |
| Login | 登入 | 建立使用者身份 |
| Register | 註冊 | 建立會員資料 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立會員註冊表單。
2. 驗證 email / 密碼格式。
3. 儲存會員資料。
4. 登入時檢查帳密。
5. 登入成功後建立身份 cookie 或 token。

```csharp
public sealed class RegisterRequest
{
    [Required, EmailAddress]
    public string Email { get; set; } = "";

    [Required, MinLength(8)]
    public string Password { get; set; } = "";
}
```

注意事項：
- 密碼不能明文存資料庫。
- 新專案建議使用 ASP.NET Core Identity 或成熟身份系統。
- 登入失敗訊息不要透露「帳號存在但密碼錯」。

### 負面例子 / 錯誤用法

錯誤做法：資料庫存 `Password = "12345678"`。

問題：資料外洩時使用者密碼全部曝光。

修正方向：使用 Identity 的 password hasher，或至少使用安全雜湊與 salt。

### 小練習

替註冊表單加入確認密碼欄位。

### 一句話總結

會員功能的第一原則是保護身份與密碼，不要自己隨手土炮安全機制。

---

## Day 16：購物中心實作四

### 這篇文章主要在講什麼

這一階段常進入購物車：使用者把商品加入暫存清單，稍後結帳。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Cart | 購物車 | 暫存欲購買商品 |
| CartItem | 購物車項目 | 商品、數量、單價 |
| Session / DB | 暫存來源 | 保存購物車狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 商品詳細頁提供加入購物車按鈕。
2. POST `AddToCart(productId, quantity)`。
3. 檢查商品存在與庫存。
4. 新增或累加購物車項目。
5. redirect 到購物車頁確認。

```csharp
public sealed record AddToCartRequest(int ProductId, int Quantity);

[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Add(AddToCartRequest request)
{
    // 參數說明：ProductId 來自商品頁，Quantity 是使用者選擇數量。
    // 副作用：真實專案會寫入購物車資料表或 session。
    if (request.Quantity <= 0) return BadRequest();

    var productExists = await _db.Products.AnyAsync(p => p.Id == request.ProductId);
    if (!productExists) return NotFound();

    TempData["Message"] = "已加入購物車";
    return RedirectToAction("Index", "Cart");
}
```

注意事項：
- 價格結帳時要重新讀資料庫，不能完全相信前端傳來的價格。
- 購物車可以存在 session、cookie、資料庫，選擇取決於登入需求與跨裝置需求。
- 數量必須驗證上限。

### 負面例子 / 錯誤用法

錯誤做法：前端傳 `price`，後端直接使用該價格結帳。

問題：使用者可竄改價格。

修正方向：前端只傳商品 id 與數量，價格由後端查詢。

### 小練習

實作同商品重複加入時累加數量。

### 一句話總結

購物車是使用者意圖的暫存，真正可信的商品價格與庫存仍要由後端決定。

---

## Day 17：購物中心實作五

### 這篇文章主要在講什麼

購物車需要顯示明細、修改數量、移除項目，讓使用者在結帳前確認。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Cart Summary | 購物車明細 | 顯示商品、數量、小計 |
| Update Quantity | 修改數量 | 調整購買數量 |
| Remove Item | 移除項目 | 從購物車刪除商品 |

### 完整實作流程、範例與注意事項

完整流程：

1. 查詢購物車項目。
2. 轉成 `CartViewModel`。
3. View 顯示單價、數量、小計。
4. POST 修改數量。
5. POST 移除商品。

```csharp
public sealed record CartItemViewModel(string ProductName, int Quantity, decimal UnitPrice)
{
    public decimal Subtotal => Quantity * UnitPrice;
}
```

注意事項：
- 小計可在 ViewModel 計算，但最終訂單金額仍要在後端服務重新計算。
- 修改數量要檢查庫存。
- 移除操作建議使用 POST。

### 負面例子 / 錯誤用法

錯誤做法：購物車總金額由前端 JavaScript 算完後送給後端保存。

問題：金額可被竄改，也可能因小數與折扣規則不一致出錯。

修正方向：後端以商品資料與數量重新計算。

### 小練習

加入購物車總數量與總金額顯示。

### 一句話總結

購物車畫面是給使用者確認，結帳金額一定要由後端重新計算。

---

## Day 18：購物中心實作六

### 這篇文章主要在講什麼

這一階段會進入訂單建立：把購物車轉成不可隨意變動的訂單紀錄。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Order | 訂單 | 保存一次購買交易 |
| OrderItem | 訂單明細 | 保存商品、數量、成交單價 |
| Checkout | 結帳 | 從購物車建立訂單 |

### 完整實作流程、範例與注意事項

完整流程：

1. 使用者按下結帳。
2. 後端讀取購物車。
3. 檢查庫存與商品狀態。
4. 建立 Order 與 OrderItem。
5. 清空購物車並顯示訂單結果。

```csharp
public sealed class Order
{
    public int Id { get; set; }
    public DateTimeOffset CreatedAt { get; set; }
    public decimal TotalAmount { get; set; }
    public List<OrderItem> Items { get; set; } = new();
}

public sealed class OrderItem
{
    public int ProductId { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}
```

注意事項：
- 訂單明細要保存成交當下的單價，不要只保存商品 id。
- 建立訂單與扣庫存應在交易中完成。
- 結帳前要再次檢查商品是否下架。

### 負面例子 / 錯誤用法

錯誤做法：訂單只存商品 id，顯示歷史訂單時再查目前商品價格。

問題：商品改價後歷史訂單金額會變。

修正方向：OrderItem 保存成交單價與商品名稱快照。

### 小練習

替 Order 加上 `Status`，例如 Pending、Paid、Cancelled。

### 一句話總結

訂單是交易快照，必須保存當下價格、數量與狀態。

---

## Day 19：購物中心實作七

### 這篇文章主要在講什麼

訂單建立後，需要會員查詢自己的訂單，管理員也可能需要查看所有訂單。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| My Orders | 我的訂單 | 會員查詢自己的訂單 |
| Admin Orders | 訂單管理 | 後台查詢與處理訂單 |
| Authorization | 權限 | 限制可見資料範圍 |

### 完整實作流程、範例與注意事項

完整流程：

1. 會員登入後取得 user id。
2. 查詢該 user id 的訂單。
3. 顯示訂單列表與狀態。
4. 點入訂單詳細。
5. 確認使用者不能看別人的訂單。

```csharp
[Authorize]
public async Task<IActionResult> Details(int id)
{
    var userId = User.FindFirstValue(ClaimTypes.NameIdentifier);

    var order = await _db.Orders
        .Where(o => o.Id == id && o.UserId == userId)
        .Select(o => new OrderDetailViewModel(o.Id, o.TotalAmount, o.CreatedAt))
        .SingleOrDefaultAsync();

    return order is null ? NotFound() : View(order);
}
```

注意事項：
- 查詢條件必須包含目前使用者。
- 管理員與會員查詢邏輯要分開。
- 訂單詳細頁也要檢查權限，不只列表頁。

### 負面例子 / 錯誤用法

錯誤做法：`/Orders/Details/100` 只用 id 查訂單。

問題：使用者改 URL 就可能看到別人的訂單。

修正方向：查詢時加上 user id 或授權 policy。

### 小練習

讓會員只能查自己的訂單，管理員可以查所有訂單。

### 一句話總結

資料授權要跟查詢條件綁在一起，不能只靠頁面入口控制。

---

## Day 20：購物中心實作八

### 這篇文章主要在講什麼

購物網站實作收尾通常會整理流程、修正 UI、補齊後台或訂單狀態。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Admin Flow | 後台流程 | 管理商品與訂單 |
| User Flow | 前台流程 | 瀏覽、加入購物車、結帳 |
| End-to-end Check | 完整流程檢查 | 確認功能串起來 |

### 完整實作流程、範例與注意事項

完整流程：

1. 從首頁瀏覽商品。
2. 進入商品詳細頁。
3. 加入購物車。
4. 調整購物車數量。
5. 結帳建立訂單，並查詢訂單。

```text
驗收 checklist：
1. 未登入使用者能瀏覽商品。
2. 會員能加入購物車。
3. 商品不存在時回 404。
4. 購物車數量不可小於 1。
5. 訂單金額由後端重新計算。
```

注意事項：
- 完成功能後要做端到端測試，不只看單一頁。
- 錯誤情境要測：無商品、庫存不足、未登入、權限不足。
- 可以用測試資料固定驗收流程。

### 負面例子 / 錯誤用法

錯誤做法：只測「正常買一個商品」。

問題：實務 bug 常出現在邊界情境。

修正方向：補測無庫存、重複加入、未登入結帳、越權查訂單。

### 小練習

寫一份購物網站手動驗收 checklist。

### 一句話總結

購物網站不是單頁功能集合，而是一條從瀏覽到訂單完成的完整使用者流程。

---

## Day 21：網站部署一：Azure 雲端帳號建立

### 這篇文章主要在講什麼

原文開始部署篇，介紹 Azure 帳號與雲端服務。2026 實務上仍可用 Azure App Service，但要注意成本與資源區域。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Cloud Provider | Azure | 提供部署環境 |
| App Service | 網站服務 | 執行 Web App |
| Resource Group | 資源群組 | 管理相關雲端資源 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立 Azure 帳號。
2. 建立 Resource Group。
3. 建立 App Service Plan。
4. 建立 Web App。
5. 部署後開啟公開 URL 驗證。

```text
部署前檢查：
1. 確認訂閱與計費方式。
2. 選擇靠近使用者的 region。
3. 設定最低可接受的 App Service Plan。
4. 不要把正式密碼寫進原始碼。
5. 部署後檢查 log 與 health endpoint。
```

注意事項：
- 免費或低階方案可能有休眠、效能與限制。
- 雲端資源不用時要關閉或刪除，避免持續計費。
- 正式環境要有監控與 log。

### 負面例子 / 錯誤用法

錯誤做法：測試完忘記刪 Azure SQL 或 App Service Plan。

問題：雲端資源可能持續收費。

修正方向：練習用資源建立標籤，結束後清理 Resource Group。

### 小練習

建立一份部署資源清單，記錄每個資源用途與是否會計費。

### 一句話總結

部署到雲端前，要同時理解技術流程與成本邊界。

---

## Day 22：網站部署二：Azure 資料庫使用

### 這篇文章主要在講什麼

原文說明 Azure SQL Database 的建立與連線，讓網站資料庫從本機移到雲端。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Azure SQL | Azure 資料庫 | 雲端 SQL Server |
| Connection String | 連線字串 | Web App 連接資料庫所需設定 |
| Firewall Rule | 防火牆規則 | 控制誰能連資料庫 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立 Azure SQL Server 與 Database。
2. 設定防火牆允許開發機或 App Service 連線。
3. 在 Web App 設定 connection string。
4. 執行 migration 或部署資料表。
5. 測試網站能讀寫雲端資料庫。

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=tcp:<server>.database.windows.net,1433;Database=<db>;User ID=<user>;Password=<password>;Encrypt=True;"
  }
}
```

注意事項：
- 上方 JSON 只是格式示意，不要把正式密碼 commit。
- 正式環境建議用 App Service 設定、Key Vault 或 managed identity。
- 防火牆不要長期開放所有 IP。

### 負面例子 / 錯誤用法

錯誤做法：把正式 Azure SQL 帳密放進 GitHub。

問題：憑證外洩後資料庫可能被入侵或產生費用。

修正方向：立刻 rotate password，改用環境變數或 Key Vault。

### 小練習

把本機與正式環境 connection string 分開設定。

### 一句話總結

雲端資料庫部署的重點不是只連得上，而是連線設定、權限與憑證管理要安全。

---

## Day 23：Entity Framework 連線字串加密

### 這篇文章主要在講什麼

原文介紹 EF 連線字串加密。2026 實務更常見做法是避免把秘密放在檔案：本機用 User Secrets，正式環境用環境變數或 Key Vault。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Connection String | 連線字串 | 資料庫位置與認證資訊 |
| Secret | 機密設定 | 密碼、token、key |
| Secret Store | User Secrets / Key Vault | 安全保存機密設定 |

### 完整實作流程、範例與注意事項

完整流程：

1. 本機開發使用 User Secrets。
2. CI/CD 使用安全變數。
3. 正式環境使用環境變數或 Key Vault。
4. appsettings 只保留非機密預設值。
5. 定期檢查 Git history 沒有秘密。

```powershell
# 範例用途：本機開發設定 connection string，不寫入 appsettings.json。
# 副作用：寫入目前使用者的 secret store，不會進入 Git。
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=localhost;Database=Shop;Trusted_Connection=True;TrustServerCertificate=True"
```

注意事項：
- 加密檔案不等於安全管理，解密 key 放一起仍然危險。
- 機密曾經 commit 過，就算刪除檔案也要 rotate。
- 不同環境應使用不同帳號與權限。

### 負面例子 / 錯誤用法

錯誤做法：在 `appsettings.Production.json` 放正式資料庫密碼並 commit。

問題：repo 權限擴散後密碼也擴散。

修正方向：移到 secret store，並更換已外洩密碼。

### 小練習

檢查自己的 repo 是否有 `Password=`、`User ID=`、`ApiKey` 等字樣。

### 一句話總結

機密管理的最佳做法是不要把秘密放進原始碼，而不是只把秘密藏起來。

---

## Day 24：Partial View

### 這篇文章主要在講什麼

原文介紹 Partial View，把重複出現的畫面片段抽出來重用。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Partial View | 部分檢視 | 可重用的 Razor 片段 |
| Parent View | 父 View | 呼叫 partial 的頁面 |
| View Component | View Component | ASP.NET Core 中較適合含邏輯的畫面元件 |

### 完整實作流程、範例與注意事項

完整流程：

1. 找出重複 UI，例如商品卡片。
2. 建立 `_ProductCard.cshtml`。
3. 定義該 partial 的 model。
4. 父 View 對每筆商品呼叫 partial。
5. 確認 partial 不依賴隱藏的 ViewBag。

```cshtml
@model ProductCardViewModel

<article class="card">
    <div class="card-body">
        <h2 class="h5">@Model.Name</h2>
        <p>@Model.Price</p>
    </div>
</article>
```

注意事項：
- Partial 適合純顯示片段。
- 如果元件需要自己查資料，ASP.NET Core 可考慮 View Component。
- 命名通常用 `_` 開頭表示 partial。

### 負面例子 / 錯誤用法

錯誤做法：Partial 依賴 `ViewBag.CurrentUserRole` 才能顯示。

問題：呼叫者很難知道 partial 需要哪些資料。

修正方向：把需要的資料放進 partial ViewModel。

### 小練習

把商品卡片從列表頁抽成 `_ProductCard.cshtml`。

### 一句話總結

Partial View 用來重用畫面片段，但資料需求仍要明確。

---

## Day 25：路由 Route

### 這篇文章主要在講什麼

原文介紹 URL 如何對應 Controller 與 Action。路由決定使用者打哪個網址會進到哪段程式。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Route | 路由 | URL 對應規則 |
| Controller Route Value | controller | 決定 Controller |
| Action Route Value | action | 決定 Action |
| Route Parameter | id | URL 中的變動值 |

### 完整實作流程、範例與注意事項

完整流程：

1. 使用預設路由 `{controller=Home}/{action=Index}/{id?}`。
2. 建立 `ProductsController.Detail(int id)`。
3. 測試 `/Products/Detail/1`。
4. 需要更漂亮 URL 時使用 attribute routing。
5. 加上 route constraint 避免錯誤輸入。

```csharp
[Route("products")]
public class ProductsController : Controller
{
    [HttpGet("{id:int}")]
    public IActionResult Detail(int id)
    {
        return View();
    }
}
```

注意事項：
- URL 是產品介面的一部分，不要隨意變動。
- route constraint 可減少不合法 request 進入 action。
- SEO 與可讀性都會受到路由設計影響。

### 負面例子 / 錯誤用法

錯誤做法：商品詳細頁 URL 是 `/Home/Page?id=123&type=product&mode=detail`。

問題：可讀性差，也不利維護與分享。

修正方向：使用 `/products/123` 或 `/products/{slug}`。

### 小練習

建立 `/categories/{categoryId:int}/products` 顯示分類商品。

### 一句話總結

路由是 URL 與程式入口的契約，設計得清楚會讓系統更好理解。

---

## Day 26：各種 ActionResult

### 這篇文章主要在講什麼

原文介紹不同 ActionResult，例如 View、Redirect、Content、Json、File、NotFound。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| ViewResult | View | 回傳 HTML 頁面 |
| RedirectResult | Redirect / RedirectToAction | 要瀏覽器改去其他 URL |
| JsonResult | Json | 回傳 JSON |
| NotFoundResult | HttpNotFound / NotFound | 回傳 404 |
| FileResult | File | 回傳檔案 |

### 完整實作流程、範例與注意事項

完整流程：

1. 顯示頁面用 `View(model)`。
2. 表單成功用 `RedirectToAction`。
3. 找不到資料用 `NotFound()`。
4. AJAX / API 資料用 `Json()` 或 API Controller。
5. 下載檔案用 `File()`。

```csharp
public async Task<IActionResult> Detail(int id)
{
    var product = await _db.Products.FindAsync(id);

    if (product is null)
    {
        // NotFoundResult：告訴使用者與搜尋引擎此資源不存在。
        return NotFound();
    }

    return View(product);
}
```

注意事項：
- 不要用 200 OK 顯示「找不到」，應回正確 HTTP status。
- 表單成功後 redirect。
- JSON API 與 MVC View 可以分開 controller 管理。

### 負面例子 / 錯誤用法

錯誤做法：找不到商品時 `return View("Error")`，但 HTTP status 仍是 200。

問題：監控、SEO、使用者代理都會誤判成功。

修正方向：回 `NotFound()` 或設定正確 status code。

### 小練習

建立 action，找不到商品時回 404，成功時回 View。

### 一句話總結

ActionResult 不只是回畫面，而是在表達這次 HTTP request 的結果。

---

## Day 27：自訂 Helper

### 這篇文章主要在講什麼

原文介紹自訂 Helper，把常用 HTML 產生邏輯封裝起來。2026 在 ASP.NET Core 中可考慮 Tag Helper 或 View Component。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Custom Helper | 自訂 Helper | 重用 HTML 產生邏輯 |
| Tag Helper | Tag Helper | ASP.NET Core 的標籤擴充 |
| View Component | View Component | 可包含邏輯的 UI 元件 |

### 完整實作流程、範例與注意事項

完整流程：

1. 找出重複 HTML。
2. 判斷只是格式化還是需要資料查詢。
3. 純格式化可用 helper。
4. 需要資料與邏輯可用 View Component。
5. 寫測試或至少建立多個頁面驗證。

```csharp
public static class PriceFormatHelper
{
    // 範例用途：集中商品價格顯示格式。
    // 參數說明：price 是資料庫或服務計算後的價格。
    // 回傳結果：回傳已格式化的台幣文字。
    public static string ToTwd(this decimal price)
    {
        return $"NT$ {price:N0}";
    }
}
```

注意事項：
- Helper 適合簡單、純輸入輸出的顯示邏輯。
- 不要在 Helper 裡查資料庫。
- 複雜元件應用 View Component 或 Partial + ViewModel。

### 負面例子 / 錯誤用法

錯誤做法：Helper 裡面讀取 DbContext 查使用者購物車。

問題：隱藏資料依賴，測試與效能都難控。

修正方向：Controller / View Component 負責準備資料，Helper 只做格式化。

### 小練習

寫一個 `ToTwd()` extension，讓價格統一顯示。

### 一句話總結

自訂 Helper 要保持單純，適合格式化，不適合承載商業流程。

---

## Day 28：JSON 資料格式

### 這篇文章主要在講什麼

原文介紹 JSON 格式、物件與陣列，以及如何把外部 JSON 轉成 C# 類別。2026 可優先使用 `System.Text.Json`，必要時再用 `Newtonsoft.Json`。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| JSON Object | `{}` | 一筆物件資料 |
| JSON Array | `[]` | 多筆資料集合 |
| Deserialize | 反序列化 | JSON 字串轉 C# 物件 |
| HttpClient | HttpClient | 呼叫外部 HTTP API |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認外部 API URL。
2. 建立 C# DTO。
3. 用 HttpClient 送 GET。
4. 用 `ReadFromJsonAsync<T>()` 反序列化。
5. 檢查 null、HTTP status、欄位型別。

```csharp
public sealed record BikeStationDto(string sna, int sbi, int bemp);

public async Task<IReadOnlyList<BikeStationDto>> GetStationsAsync(HttpClient client)
{
    // 參數說明：client 由 DI 建立，base address 與 timeout 可集中設定。
    // 回傳結果：回傳 YouBike 站點清單；失敗時丟出 HTTP 例外。
    var url = "https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json";
    var stations = await client.GetFromJsonAsync<List<BikeStationDto>>(url);
    return stations ?? [];
}
```

注意事項：
- 外部 API 可能失敗、變慢或改格式。
- 數字、日期、null 欄位要小心型別。
- 不要每次 request 都無限制呼叫外部 API，可考慮快取。

### 負面例子 / 錯誤用法

錯誤做法：假設外部 API 永遠成功，直接使用回傳資料。

問題：API 失敗時頁面爆掉，使用者看到 500。

修正方向：檢查 status code、timeout、null，並提供 fallback。

### 小練習

把 YouBike API 取回後，只顯示可借車數大於 0 的站點。

### 一句話總結

JSON 是系統交換資料的通用格式，但外部資料永遠要驗證與容錯。

---

## Day 29：PagedList 套件使用

### 這篇文章主要在講什麼

原文用 PagedList 將大量 YouBike 資料分頁顯示，避免同一頁載入太多資料。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Pagination | 分頁 | 每次只顯示部分資料 |
| Page | page | 目前頁碼 |
| PageSize | pageSize | 每頁筆數 |
| PagedList | PagedList | 分頁模型或套件 |

### 完整實作流程、範例與注意事項

完整流程：

1. Controller 接收 `page`。
2. 修正小於 1 的頁碼。
3. 設定 `pageSize`。
4. 查詢時使用 `Skip` / `Take`。
5. View 顯示頁碼與上一頁下一頁。

```csharp
public async Task<IActionResult> Index(int page = 1)
{
    // 參數說明：page 來自 query string，例如 /Products?page=2。
    // 回傳結果：只回傳目前頁面的商品資料。
    const int pageSize = 20;
    var currentPage = Math.Max(page, 1);

    var products = await _db.Products
        .OrderBy(p => p.Id)
        .Skip((currentPage - 1) * pageSize)
        .Take(pageSize)
        .Select(p => new ProductCardViewModel(p.Id, p.Name, p.Price, p.Category.Name))
        .ToListAsync();

    return View(products);
}
```

注意事項：
- 資料庫分頁一定要有穩定排序。
- `Skip` 很大時可能效能變差，可進階使用 keyset pagination。
- 頁碼、每頁筆數都要限制，避免使用者要求超大資料量。

### 負面例子 / 錯誤用法

錯誤做法：先 `ToListAsync()` 取全部商品，再在記憶體分頁。

問題：資料多時記憶體與資料庫傳輸都浪費。

修正方向：在資料庫查詢階段使用 `Skip` / `Take`。

### 小練習

加入總筆數，讓畫面顯示「第 2 頁 / 共 10 頁」。

### 一句話總結

分頁的目標是控制一次載入的資料量，查詢端就要完成分頁。

---

## Day 30：結語

### 這篇文章主要在講什麼

原文分享完賽心得與後續學習方向，例如 Web API、ASP.NET Core、SQL、JavaScript。從 2026 角度看，這些方向仍然正確，只是優先順序可更清楚。

### 2026 學習路線建議

1. 先用 ASP.NET Core MVC 做一個可部署的小系統。
2. 補 EF Core、SQL、資料庫正規化與查詢效能。
3. 學 ASP.NET Core Web API，理解 MVC 頁面與 API 的差異。
4. 補身份驗證、授權、OWASP 常見風險。
5. 學部署、環境變數、logging、CI/CD。
6. 補 JavaScript / TypeScript，至少能處理前端互動。

### 完整實作流程、範例與注意事項

完整流程：

1. 選一個 Side Project 題目，例如簡易購物站、書籍管理、庫存系統。
2. 建立需求清單，限制在 2 到 4 週可完成。
3. 完成 MVC 前台與後台。
4. 接上資料庫、登入、基本權限。
5. 部署到雲端並寫 README。

```text
作品集 README 建議包含：
1. 專案目標。
2. 技術棧與版本。
3. 本機啟動方式。
4. 資料庫 migration 方式。
5. 測試帳號或測試資料。
6. 已知限制與後續改進。
```

注意事項：
- 作品集不需要什麼都做，但要能穩定跑完核心流程。
- 比起堆功能，更重要的是資料流、錯誤處理與部署文件清楚。
- 面試時能說明「為什麼這樣設計」比只說「我照教學做」更有價值。

### 負面例子 / 錯誤用法

錯誤做法：跟完 30 天後只截圖，沒有整理自己的專案 README 與設計說明。

問題：面試或回頭維護時，很難證明自己理解架構。

修正方向：用自己的話整理架構、資料表、功能流程與踩雷修正。

### 小練習

把本系列改寫成自己的 ASP.NET Core MVC 作品，並部署一版 demo。

### 一句話總結

學完教學只是開始，能獨立重做、部署、說明設計取捨，才是真正把 MVC 學會。

---

## 跨 Day 實務總結

### 適合使用 MVC 的情境

- 後台管理系統。
- 需要伺服器端渲染 HTML 的企業網站。
- 表單、CRUD、權限、列表、報表為主的系統。
- 團隊熟悉 C# 與 Razor，前端互動不需要 SPA 複雜度。

### 不適合硬套 MVC 的情境

- 高互動前端應用，前端狀態非常複雜，可能更適合 SPA + API。
- 純 API 服務，直接用 ASP.NET Core Web API 更清楚。
- 即時協作或即時訊息場景，可能需要 SignalR 或事件架構。
- 很小的靜態網站，不需要後端 MVC。

### Junior 常見誤解

- 誤解：MVC 就是資料夾分成 Models、Views、Controllers。
  - 修正：真正重點是責任分離，不是資料夾名稱。
- 誤解：Controller 可以寫所有邏輯。
  - 修正：Controller 應該薄，商業邏輯放 Service / Domain。
- 誤解：ViewModel 跟 Entity 可以永遠共用。
  - 修正：ViewModel 是畫面需求，Entity 是資料庫結構，兩者變動理由不同。
- 誤解：前端驗證通過就安全。
  - 修正：後端永遠要再驗證。
- 誤解：能部署就完成了。
  - 修正：還要有設定管理、log、錯誤處理、備份與成本控管。

## 最小可練習專案規格

### 實作任務情境

做一個「小型商品管理與購物車」網站，包含商品列表、商品詳細、加入購物車、結帳建立訂單、會員查詢訂單。

### 操作前檢查

- 已安裝 .NET 10 SDK 或團隊指定版本。
- 已安裝 Visual Studio 2022 / VS Code。
- 已準備 SQL Server、LocalDB、Docker SQL Server 或 SQLite。
- 已理解 C# 類別、List、LINQ、async/await 基礎。

### Step-by-step 實作

1. 建立 ASP.NET Core MVC 專案。
2. 建立 Product、CartItem、Order、OrderItem。
3. 建立 DbContext 與 migration。
4. 建立商品列表與詳細頁。
5. 建立購物車加入、修改、移除流程。
6. 建立結帳流程，保存訂單快照。
7. 建立會員訂單查詢頁。
8. 加上後台權限。
9. 加上分頁與基本搜尋。
10. 部署到 Azure App Service 或其他平台。

### 做完後檢查

- 首頁能顯示商品。
- 商品詳細不存在時回 404。
- 表單驗證錯誤會回原頁並顯示訊息。
- 購物車不能加入負數數量。
- 訂單金額由後端計算。
- 未登入不能看會員訂單。
- 使用者不能查別人的訂單。
- 正式環境沒有把 connection string commit 到 Git。

### 最後一句話

這個系列最值得帶走的不是 MVC5 的某個 API，而是「用 MVC 把一個網站需求拆成可理解、可維護、可部署的工程流程」。
