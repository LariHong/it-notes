# 淺談資料庫、ASP.NET 與 C# 入門：2026 更新整理

來源系列：[淺談資料庫&ASP.net&C# 入門](https://ithelp.ithome.com.tw/users/20119925/ironman/2276)，作者：捲毛蔡，iT 邦幫忙第 11 屆鐵人賽，2019-09-02 至 2019-10-09，共 36 篇。

這份筆記不是逐字重寫原文，而是把系列內容整理成「今天仍適合初學者使用」的版本。原文很適合建立 SQL Server、資料表、基本 SQL、ASP.NET Web Forms 連資料庫的直覺；但 2019 到 2026 之間，.NET、ASP.NET、SQL Server 與安全實務都有明顯演進，所以本文會把過時或需要補強的地方標出來。

## 先講結論

- 原系列的核心學習順序仍然有價值：資料庫概念 -> SQL 基礎 -> SQL Server 操作 -> ASP.NET 讀寫資料庫 -> 小專案實作。
- 若是全新專案，今日建議優先使用 `.NET 10 LTS`、`ASP.NET Core`、`Razor Pages / MVC / Minimal APIs / Blazor`，不要把 ASP.NET Web Forms 當成新專案首選。
- Web Forms 仍屬於 .NET Framework 世界。Microsoft 文件明確指出 ASP.NET Web Forms 只存在於 .NET Framework，ASP.NET Core 不能用 Web Forms。
- ADO.NET 仍可學，但新專案建議使用 `Microsoft.Data.SqlClient` 或 Entity Framework Core，而不是只停在 `System.Data.SqlClient`。
- 任何把使用者輸入串進 SQL 字串的寫法都要避免。今天應以參數化查詢、ORM、最小權限、輸入驗證與輸出編碼作為基本安全線。
- SQL Server 版本已從原文的 SQL Server 2017 時代推進到 SQL Server 2025；安裝、SSMS、Azure SQL、Docker、本機開發與雲端資料庫的選擇都比當年多。

## 2026 今日建議版本

| 主題 | 2019 原文常見做法 | 2026 建議 |
| --- | --- | --- |
| .NET | .NET Framework / Web Forms | 新專案用 .NET 10 LTS；既有 Web Forms 才留在 .NET Framework |
| Web 框架 | ASP.NET Web Forms | ASP.NET Core：Razor Pages、MVC、Minimal APIs、Blazor |
| SQL Server | SQL Server 2017 Express | SQL Server 2025 Developer/Express、Azure SQL、容器化 SQL Server |
| SQL Client | `System.Data.SqlClient` | `Microsoft.Data.SqlClient` 或 EF Core |
| 資料存取 | `DataReader`、`DataSet`、`SqlDataAdapter` | 小型範例可用 ADO.NET；正式系統優先分層、非同步、參數化、EF Core |
| 分頁 | `ROW_NUMBER()` 自製分頁 | `OFFSET ... FETCH`、Keyset pagination，或 ORM 分頁 |
| 登入驗證 | FormsAuthentication、Cookie Ticket | ASP.NET Core Identity、OpenID Connect、OAuth 2.1 / OIDC、MFA |
| 安全 | ValidateRequest、手動 confirm | 參數化查詢、CSRF 防護、XSS 輸出編碼、密碼雜湊、HTTPS、最小權限 |

## 系列目錄

| Day | 原文主題 | 原文連結 | 今日閱讀重點 |
| --- | --- | --- | --- |
| 1 | 嗨嗨鐵人賽-前言 | [連結](https://ithelp.ithome.com.tw/articles/10213191?sc=rss.iron) | 了解系列目標即可 |
| 2 | 資料庫簡單介紹 | [連結](https://ithelp.ithome.com.tw/articles/10213407?sc=rss.iron) | 補上 RDBMS、NoSQL、雲端資料庫的差異 |
| 3 | SQL 語法介紹 | [連結](https://ithelp.ithome.com.tw/articles/10213596?sc=rss.iron) | DDL、DML、DCL 分類仍重要 |
| 4 | SQL Server 下載安裝 | [連結](https://ithelp.ithome.com.tw/articles/10213758?sc=rss.iron) | 改看 SQL Server 2025 / SSMS 最新安裝方式 |
| 5 | 主索引鍵與外部索引鍵 | [連結](https://ithelp.ithome.com.tw/articles/10213896?sc=rss.iron) | 關聯式資料庫核心，仍很重要 |
| 6 | char、varchar、nchar、nvarchar | [連結](https://ithelp.ithome.com.tw/articles/10213922?sc=rss.iron) | 補 Unicode、UTF-8 collation、長度限制 |
| 7 | 手動建立資料表 | [連結](https://ithelp.ithome.com.tw/articles/10214308?sc=rss.iron) | 建表概念有效，建議同步學 migration |
| 8 | 手動存入資料 | [連結](https://ithelp.ithome.com.tw/articles/10214381?sc=rss.iron) | 練習即可，正式系統用 migrations / seed |
| 9 | SQL Server 驗證使用者 | [連結](https://ithelp.ithome.com.tw/articles/10214386?sc=rss.iron) | 改補最小權限、Entra ID、秘密管理 |
| 10 | SQL 語法小精靈 | [連結](https://ithelp.ithome.com.tw/articles/10214682?sc=rss.iron) | 可輔助學習，但不要依賴 GUI 產 SQL |
| 11 | 匯入 Excel | [連結](https://ithelp.ithome.com.tw/articles/10214762?sc=rss.iron) | 正式資料匯入要驗證欄位、型別、交易 |
| 12 | ASP.NET 與資料庫聯繫 | [連結](https://ithelp.ithome.com.tw/articles/10214990?sc=rss.iron) | 新專案改看 ASP.NET Core + DI + config |
| 13 | DataReader | [連結](https://ithelp.ithome.com.tw/articles/10215075?sc=rss.iron) | 了解 forward-only reader；補 async / using |
| 14 | DataSet | [連結](https://ithelp.ithome.com.tw/articles/10215263?sc=rss.iron) | 適合離線資料快取；Web API 常不需要 |
| 15 | SELECT | [連結](https://ithelp.ithome.com.tw/articles/10214938?sc=rss.iron) | SQL 查詢基本功 |
| 16 | JOIN | [連結](https://ithelp.ithome.com.tw/articles/10215741?sc=rss.iron) | 必學，補 execution plan 與索引觀念 |
| 17 | UNION / INTERSECT / EXCEPT | [連結](https://ithelp.ithome.com.tw/articles/10216761?sc=rss.iron) | 集合思維很重要 |
| 18 | ORDER BY | [連結](https://ithelp.ithome.com.tw/articles/10217026?sc=rss.iron) | 分頁必須搭配穩定排序 |
| 19 | 聚合函數 | [連結](https://ithelp.ithome.com.tw/articles/10218055?sc=rss.iron) | COUNT/AVG/MAX/MIN/SUM 仍常用 |
| 20 | GROUP BY | [連結](https://ithelp.ithome.com.tw/articles/10218537?sc=rss.iron) | 補 HAVING、GROUPING、Window Functions |
| 21 | IN / NOT IN | [連結](https://ithelp.ithome.com.tw/articles/10219093?sc=rss.iron) | 注意 NULL；可比較 EXISTS |
| 22 | 子查詢 | [連結](https://ithelp.ithome.com.tw/articles/10219497?sc=rss.iron) | 補 CTE、APPLY、EXISTS |
| 23 | INSERT | [連結](https://ithelp.ithome.com.tw/articles/10220002?sc=rss.iron) | 補交易、批次匯入、預設值 |
| 24 | UPDATE | [連結](https://ithelp.ithome.com.tw/articles/10220040?sc=rss.iron) | 必加 WHERE；正式環境先 SELECT 驗證 |
| 25 | DELETE | [連結](https://ithelp.ithome.com.tw/articles/10220412?sc=rss.iron) | 補軟刪除、交易、備份 |
| 26 | 數學運算子與函數 | [連結](https://ithelp.ithome.com.tw/articles/10220979?sc=rss.iron) | 基礎有效，留意型別轉換 |
| 27 | 檔案資料匯入資料庫 | [連結](https://ithelp.ithome.com.tw/articles/10221928?sc=rss.iron) | 今日要補 CSV parser、檔案驗證、防毒與批次 |
| 28 | 留言板上篇 | [連結](https://ithelp.ithome.com.tw/articles/10222342?sc=rss.iron) | 小專案概念好，但安全要重做 |
| 29 | 留言板中篇 | [連結](https://ithelp.ithome.com.tw/articles/10222752?sc=rss.iron) | 補 XSS、CSRF、輸出編碼 |
| 30 | 留言板下篇 | [連結](https://ithelp.ithome.com.tw/articles/10223123?sc=rss.iron) | 補分層架構與測試 |
| 31 | GridView 刪除資料 | [連結](https://ithelp.ithome.com.tw/articles/10224588?sc=rss.iron) | 今日改用 controller/service/repository 或 EF Core |
| 32 | FormsAuthentication | [連結](https://ithelp.ithome.com.tw/articles/10224507?sc=rss.iron) | 新專案改用 ASP.NET Core Identity/OIDC |
| 33 | 暫存表與 CTE | [連結](https://ithelp.ithome.com.tw/articles/10225120?sc=rss.iron) | 仍實用，補效能與 scope 差異 |
| 34 | ROW_NUMBER() | [連結](https://ithelp.ithome.com.tw/articles/10225653?sc=rss.iron) | Window Functions 仍重要 |
| 35 | 使用者控制項分頁 | [連結](https://ithelp.ithome.com.tw/articles/10225633?sc=rss.iron) | Web Forms 做法過時；學概念即可 |
| 36 | 找出最新日期所有資料 | [連結](https://ithelp.ithome.com.tw/articles/10226348?sc=rss.iron) | 改用日期範圍、window function 或 TOP WITH TIES |

## 學習路線重整

### 第一階段：資料庫與 SQL 基礎

先讀 Day 2、3、5、6、15 到 22。目標不是記住所有語法，而是建立這幾件事：

- 資料庫是用表格、欄位、列與關聯來描述資料。
- Primary Key 用來唯一識別一筆資料；Foreign Key 用來維持資料關聯。
- SELECT、WHERE、ORDER BY、JOIN、GROUP BY 是日常查詢的核心。
- 聚合函數、子查詢、CTE、Window Functions 是進階查詢的入口。

今日優化：學 SQL 時要一起學 execution plan、索引、交易與 NULL 行為。很多初學 bug 不是 SQL 語法不會，而是「結果看似正確，但效能或 NULL 邏輯錯了」。

### 第二階段：SQL Server 操作

Day 4、7、8、9、10、11 適合建立 SQL Server 操作感。你可以跟著做一次，但今日不建議把 GUI 操作當成唯一能力。

今日優化：

- 建表與欄位變更建議用 migration 管理，例如 EF Core migrations、DbUp、Flyway、Liquibase。
- 開發環境可用 SQL Server Developer Edition、Express、Docker 或 Azure SQL Database。
- 帳號權限不要直接給 `db_owner`。開發、部署、應用程式連線應該用不同帳號與最小權限。
- 連線字串不要寫死在程式碼，可放在 `appsettings.json`、User Secrets、環境變數或雲端 secret manager。

### 第三階段：ASP.NET 連資料庫

Day 12 到 14 是 ADO.NET 入門：連線、Command、DataReader、DataSet。這些概念仍值得學，因為它們能幫你理解底層資料存取。

今日優化：

- 新專案優先用 `Microsoft.Data.SqlClient`。
- 查詢一定要參數化。
- `SqlConnection`、`SqlCommand`、`SqlDataReader` 要用 `using` / `await using` 正確釋放。
- ASP.NET Core 專案不要在頁面事件裡直接寫 SQL，建議拆成 controller/page handler -> service -> repository/data access。
- 若資料模型穩定，EF Core 能降低 CRUD 樣板程式；若查詢高度客製或效能敏感，可混用 Dapper / ADO.NET。

### 第四階段：留言板與登入

Day 28 到 32 是小專案實作，適合把資料庫、頁面、CRUD 串起來。但這些章節也最需要現代化。

今日優化：

- 留言內容顯示時要做 HTML encoding，避免 XSS。
- 表單送出要有 CSRF 防護。
- 刪除資料要確認授權，不是只有前端 confirm。
- 登入不要手寫密碼儲存、Cookie ticket 與授權流程；新專案用 ASP.NET Core Identity 或外部身分提供者。
- 密碼必須使用專門的 password hasher，不可用一般 hash 函式自己做。

## 重要主題更新

### ASP.NET Web Forms 的定位

原文大量使用 Web Forms、GridView、SqlDataSource、使用者控制項與 FormsAuthentication。這對理解 2010 年代 ASP.NET 很有幫助，但新專案不建議從這條路開始。

Microsoft 文件指出，ASP.NET Web Forms 只在 .NET Framework 中可用，ASP.NET Core 不能用 Web Forms。若你是維護舊系統，可以繼續學 Web Forms；若你是新學習或新開發，建議直接走 ASP.NET Core。

建議替代：

- 簡單資料頁面：Razor Pages
- MVC 架構網站：ASP.NET Core MVC
- API：Minimal APIs 或 Controllers
- C# 前後端整合 UI：Blazor

### SQL Server 版本與安裝

原文以 SQL Server 2017 為主。到 2026 年，SQL Server 2025 已經是最新主線之一，Microsoft 的版本歷史頁列出 SQL Server 2025、2022 等仍支援版本與更新。初學者可以用 Developer Edition 或 Express；公司環境則常見 Azure SQL Database、SQL Managed Instance 或內部 SQL Server。

### `System.Data.SqlClient` 與 `Microsoft.Data.SqlClient`

原文使用的是當年常見的 `System.Data.SqlClient`。今日 Microsoft 的 ADO.NET 文件建議使用 `Microsoft.Data.SqlClient` 或 Entity Framework 來存取 SQL Server。`Microsoft.Data.SqlClient` 可視為較新的 SQL Server .NET provider，支援 .NET Framework 與現代 .NET。

簡化範例：

```csharp
// 範例用途：示範「`System.Data.SqlClient` 與 `Microsoft.Data.SqlClient`」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
using Microsoft.Data.SqlClient;

var connectionString = builder.Configuration.GetConnectionString("Default");

await using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();

await using var command = new SqlCommand(
    "SELECT Id, Name FROM Customers WHERE Country = @country",
    connection);

command.Parameters.AddWithValue("@country", "Taiwan");

await using var reader = await command.ExecuteReaderAsync();
while (await reader.ReadAsync())
{
    Console.WriteLine($"{reader.GetInt32(0)} {reader.GetString(1)}");
}
```

### SQL Injection 與參數化查詢

原文有不少示範程式是教學目的，但今天閱讀時要特別小心 SQL 字串插值與字串相加。Microsoft SQL Server 文件明確提醒，任何建構 SQL 字串的程序都要檢查 SQL injection 風險，並建議使用型別安全的 SQL parameters。

不建議：

```csharp
// 範例用途：示範「SQL Injection 與參數化查詢」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var sql = $"SELECT * FROM Users WHERE Name = '{name}'";
```

建議：

```csharp
// 範例用途：示範「SQL Injection 與參數化查詢」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
await using var command = new SqlCommand(
    "SELECT * FROM Users WHERE Name = @name",
    connection);

command.Parameters.Add("@name", System.Data.SqlDbType.NVarChar, 100).Value = name;
```

### 分頁：`ROW_NUMBER()`、`OFFSET FETCH`、Keyset

Day 34、35 使用 `ROW_NUMBER()` 與 CTE 做分頁，這仍是有效技術。不過現代 SQL Server 也常用：

```sql
-- 範例用途：示範「分頁：`ROW_NUMBER()`、`OFFSET FETCH`、Keyset」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
SELECT Id, Title, CreatedAt
FROM Posts
ORDER BY CreatedAt DESC, Id DESC
OFFSET @skip ROWS FETCH NEXT @take ROWS ONLY;
```

大量資料或無限捲動更建議 Keyset pagination：

```sql
-- 範例用途：示範「分頁：`ROW_NUMBER()`、`OFFSET FETCH`、Keyset」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
SELECT TOP (@take) Id, Title, CreatedAt
FROM Posts
WHERE CreatedAt < @lastCreatedAt
   OR (CreatedAt = @lastCreatedAt AND Id < @lastId)
ORDER BY CreatedAt DESC, Id DESC;
```

重點：分頁一定要有穩定排序，否則資料可能重複或漏掉。

### 最新日期資料查詢

Day 36 想找出「最新日期的所有資料」。原文使用 `CONVERT` 把時間切成日期再比對。今日可以考慮更清楚的寫法。

若要找最大時間戳的所有資料：

```sql
-- 範例用途：示範「最新日期資料查詢」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
SELECT *
FROM Orders
WHERE CreatedAt = (SELECT MAX(CreatedAt) FROM Orders);
```

若要找最新一天的所有資料：

```sql
-- 範例用途：示範「最新日期資料查詢」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
DECLARE @latestDate date = (
    SELECT MAX(CAST(CreatedAt AS date))
    FROM Orders
);

SELECT *
FROM Orders
WHERE CreatedAt >= @latestDate
  AND CreatedAt < DATEADD(day, 1, @latestDate);
```

這樣比把欄位包在 `CONVERT(varchar(...))` 裡更容易使用索引。

## Junior 學習提醒

- 先把 SQL 寫對，再追求 ORM 或架構。ORM 不是不用學 SQL 的理由。
- GUI 精靈可以幫你開始，但正式工作要能讀懂 SQL。
- `SELECT *` 練習可以，正式 API 建議列明欄位。
- `UPDATE`、`DELETE` 執行前先用同樣 WHERE 做 `SELECT` 確認範圍。
- 資料表設計先想「這筆資料如何唯一識別」、「跟誰有關聯」、「會怎麼查詢」。
- Web 表單不是只要能存資料就完成，還要處理驗證、授權、XSS、CSRF、錯誤紀錄與交易。

## 36 篇逐篇詳細整理

下面這段才是完整的逐篇筆記版。前面的內容是系列導覽，這裡會依 Day 1 到 Day 36 把每篇文章拆成「原文重點」、「2026 更新」、「練習建議」。如果你要複習，可以直接從這一節開始讀。

### Day 1：嗨嗨鐵人賽-前言

原文重點：作者說明參賽動機，主題會以資料庫為主，偶爾穿插 ASP.NET 與 C#。這篇不是技術文，而是建立學習脈絡：用 30 天從資料庫基礎一路做到簡單 Web 應用。

2026 更新：這樣的學習路線仍然合理，但今日建議把「ASP.NET Web Forms」改成「ASP.NET Core」。如果是完全新手，順序可以改成：SQL 基礎、SQL Server 操作、C# 基礎、ASP.NET Core Razor Pages、EF Core 或 ADO.NET。

練習建議：先確認你要學的是「維護舊系統」還是「開新系統」。維護舊系統才需要深入 Web Forms；新系統請直接從 ASP.NET Core 開始。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddRazorPages();

var app = builder.Build();

app.MapRazorPages();
app.Run();
```

完整流程：

1. 建立 ASP.NET Core 空專案。
2. 註冊 Razor Pages。
3. 啟動網站。
4. 確認瀏覽器能看到頁面。
5. 再開始加入資料庫功能。

注意事項：原系列用 Web Forms 當入口；新專案建議從 ASP.NET Core Razor Pages 或 MVC 開始。先建立可執行的空專案，再逐步加入資料庫，不要一開始就把登入、CRUD、分頁全部塞進去。

### Day 2：資料庫簡單介紹

原文重點：介紹資料庫、DBMS、表格、欄位、列，以及常見資料庫系統如 MySQL、SQL Server、PostgreSQL、Oracle。也用 Excel 類比資料表，讓初學者容易理解 table、column、row。

2026 更新：關聯式資料庫仍是主流，尤其在交易、報表、企業系統、金融與後台管理上非常重要。但現代系統也常混用 NoSQL、Redis、搜尋引擎、資料湖與事件流。初學者不用一次全學，但要知道 SQL Server、PostgreSQL、MySQL 這些 RDBMS 解決的是「結構化資料、查詢、交易一致性」問題。

練習建議：拿一個訂單系統當例子，先畫出 `Customers`、`Orders`、`OrderItems`、`Products` 四張表，標出每張表的主鍵與彼此關係。你能畫出關係，SQL 才會學得穩。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public sealed class Customer
{
    public int Id { get; set; }
    public required string Name { get; set; }
    public string? Email { get; set; }
    public List<Order> Orders { get; set; } = [];
}

public sealed class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public Customer? Customer { get; set; }
    public DateTimeOffset CreatedAt { get; set; }
}
```

完整流程：

1. 先畫資料表。
2. 轉成 C# Model。
3. 標出主鍵與外鍵。
4. 補導覽屬性。
5. 確認關係是一對一、一對多或多對多。

注意事項：Model 不是資料庫本身，但它能對應資料表與關聯。`CustomerId` 是外鍵欄位，`Customer` 是導覽屬性；兩者不要混淆。

### Day 3：SQL 語法介紹

原文重點：介紹 SQL 是 Structured Query Language，並把常見語法分成 DDL、DML、DCL。DDL 包含 `CREATE`、`ALTER`、`DROP`；DML 包含 `SELECT`、`INSERT`、`UPDATE`、`DELETE`；DCL 包含 `GRANT`、`REVOKE`。

2026 更新：分類仍然正確，但現代學習時還要補 TCL，也就是交易控制，例如 `BEGIN TRANSACTION`、`COMMIT`、`ROLLBACK`。正式系統的資料異動不只是會寫 `UPDATE`，還要知道失敗時怎麼回復。

練習建議：

```sql
-- 範例用途：示範「Day 3：SQL 語法介紹」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
BEGIN TRANSACTION;

UPDATE Products
SET Stock = Stock - 1
WHERE Id = 10 AND Stock > 0;

-- 確認結果正確才 COMMIT，不正確就 ROLLBACK
ROLLBACK;
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public static class CustomerSql
{
    public const string SelectByCountry = """
        SELECT Id, Name, Email
        FROM Customers
        WHERE Country = @country
        ORDER BY Name;
        """;
}
```

完整流程：

1. 先把 SQL 分類成查詢、建立、異動與權限。
2. 放到集中位置。
3. 用參數保留輸入位置。
4. 在資料庫工具先測試再放進 C#。

注意事項：初學可以先把 SQL 寫在常數中，但正式專案要避免散落各處。更重要的是永遠保留參數，例如 `@country`，不要用字串相加組 SQL。

### Day 4：SQL Server 下載安裝步驟分享

原文重點：以 SQL Server 2017 Express 與 SSMS 為例，示範下載、安裝、開啟 SSMS 並連線到 SQL Server。

2026 更新：現在可以選 SQL Server 2025 Developer Edition、Express、Azure SQL Database，或使用 Docker 容器。Developer Edition 功能完整但只供開發測試；Express 適合輕量本機學習；Azure SQL 適合雲端練習。SSMS 仍常用，也可以搭配 Azure Data Studio 或 Visual Studio / Rider 的資料庫工具。

練習建議：本機學習可以裝 SQL Server Developer Edition + SSMS。若你怕污染電腦環境，可用 Docker 跑 SQL Server，但要先熟悉容器基本操作。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
using Microsoft.Data.SqlClient;

var connectionString = builder.Configuration.GetConnectionString("Default");

await using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();

Console.WriteLine($"Connected to {connection.Database}");
```

完整流程：

1. 安裝 SQL Server 與 SSMS。
2. 建立資料庫。
3. 設定 connection string。
4. 用 C# OpenAsync 測試。
5. 失敗時先查伺服器名稱、驗證方式與憑證。

注意事項：安裝 SQL Server 後，第一個 C# 練習就是連線測試。若出現憑證錯誤，開發環境可暫用 `TrustServerCertificate=True`，正式環境應正確設定 TLS 憑證。

### Day 5：主索引鍵與外部索引鍵

原文重點：主索引鍵用來唯一識別資料列，外部索引鍵用來建立資料表之間的關係。例如訂單表的 `CustomerId` 指向客戶表的 `Id`。

2026 更新：這篇仍然非常重要。今天還要補三個觀念：主鍵不一定要有業務意義、外鍵可幫你維持資料一致性、索引與外鍵不是同一件事。外鍵約束保證關聯有效；索引則幫助查詢效能。很多資料庫會為主鍵自動建索引，但外鍵不一定會自動有適合的索引。

練習建議：

```sql
-- 範例用途：示範「Day 5：主索引鍵與外部索引鍵」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
CREATE TABLE Customers (
    Id int IDENTITY PRIMARY KEY,
    Name nvarchar(100) NOT NULL
);

CREATE TABLE Orders (
    Id int IDENTITY PRIMARY KEY,
    CustomerId int NOT NULL,
    CreatedAt datetime2 NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT FK_Orders_Customers
        FOREIGN KEY (CustomerId) REFERENCES Customers(Id)
);
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Customer>()
        .HasKey(customer => customer.Id);

    modelBuilder.Entity<Order>()
        .HasOne(order => order.Customer)
        .WithMany(customer => customer.Orders)
        .HasForeignKey(order => order.CustomerId);
}
```

完整流程：

1. 先決定每張表的 Id。
2. 建立主鍵。
3. 建立外鍵欄位。
4. 用 EF Core Fluent API 設定關聯。
5. 產生 migration。
6. 到資料庫確認 constraint。

注意事項：EF Core 可以幫你產生外鍵，但你仍要懂資料庫關係。不要只相信 ORM，請打開 migration 或資料庫確認 constraint 是否真的建立。

### Day 6：SQL Server 資料型態 char、varchar、nchar、nvarchar

原文重點：比較固定長度與可變長度字串，以及是否支援 Unicode。`char`、`varchar` 常用於非 Unicode；`nchar`、`nvarchar` 可存中文等 Unicode 字元。

2026 更新：在台灣或任何多語系系統，預設建議用 `nvarchar`。SQL Server 後來也支援 UTF-8 collation，某些情境可用 `varchar` + UTF-8 collation 儲存 Unicode，但初學者先用 `nvarchar` 最不容易踩雷。不要濫用 `nvarchar(max)`，因為長度設計會影響驗證、索引與效能。

練習建議：姓名、標題、地址可用 `nvarchar`；狀態碼、ISO 國碼、固定格式代碼可用較短長度的 `varchar` 或 `char`。設計欄位時先想最大合理長度，而不是全部給 max。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public sealed class Product
{
    public int Id { get; set; }

    [MaxLength(100)]
    public required string Name { get; set; }

    [MaxLength(20)]
    public string? Sku { get; set; }
}
```

完整流程：

1. 根據欄位用途決定長度。
2. C# 加 validation。
3. EF Core 設定欄位長度。
4. migration 產生 nvarchar 長度。
5. 寫入超長資料測試是否被擋。

注意事項：C# 的 `string` 不會自動限制長度，資料庫的 `nvarchar(100)` 才會。Model validation 與資料庫欄位限制最好一致，避免前端通過、資料庫卻失敗。

### Day 7：SQL Server 基本操作：手動創立資料表

原文重點：在 SSMS 裡手動建立資料表與欄位，並設定主鍵與欄位型別。這是初學者熟悉資料表設計介面的好方法。

2026 更新：GUI 建表適合入門，但正式專案通常要能用 SQL script 或 migration 重現資料庫結構。因為資料庫結構需要進版控、可部署、可回滾，不應只存在某個人的 SSMS 操作記憶裡。

練習建議：同一張表請做兩次：第一次用 SSMS GUI 建，第二次改成 `CREATE TABLE` SQL。你會更清楚每個欄位、限制與關聯實際上長什麼樣。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public sealed class AppDbContext(DbContextOptions<AppDbContext> options)
    : DbContext(options)
{
    public DbSet<Customer> Customers => Set<Customer>();
    public DbSet<Order> Orders => Set<Order>();
}
```

```powershell
# 範例用途：示範「完整實作流程、Coding 例子與注意事項」中可直接在終端機執行的 PowerShell 指令。
# 參數說明：命令中的 URL、檔名、路徑、選項或環境名稱請替換成你的實際目標。
# 回傳結果 / 副作用：通常會輸出結果、讀寫檔案、下載資料，或改變目前 shell / 系統狀態。
dotnet ef migrations add InitialCreate
dotnet ef database update
```

完整流程：

1. 設計資料表。
2. 寫 C# Entity。
3. 註冊 DbContext。
4. 建立 migration。
5. database update。
6. 用 SSMS 確認資料表。

注意事項：GUI 建表適合理解概念；團隊開發要用 migration，才能讓資料庫結構跟程式碼一起版控。

### Day 8：SQL Server 基本操作：手動存入資料

原文重點：示範用 SSMS 手動輸入資料列，建立 Customers、Orders、Products、OrderDetail 等練習資料，並觀察資料如何互相關聯。

2026 更新：手動輸入很適合練習，但正式測試資料建議使用 seed script 或 migration seed。若測試資料只靠手打，別人無法重現，你也很難在新環境快速建置。

練習建議：

```sql
-- 範例用途：示範「Day 8：SQL Server 基本操作：手動存入資料」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
INSERT INTO Customers (Name)
VALUES (N'Apple'), (N'Asus'), (N'Sony');

INSERT INTO Products (Name, Price)
VALUES (N'Keyboard', 1200), (N'Mouse', 600);
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
db.Customers.AddRange(
    new Customer { Name = "Apple", Email = "apple@example.com" },
    new Customer { Name = "Asus", Email = "asus@example.com" }
);

await db.SaveChangesAsync();
```

完整流程：

1. 準備 seed 資料。
2. 檢查是否已存在。
3. 新增資料。
4. SaveChangesAsync。
5. 查詢確認資料是否真的寫入。

注意事項：測試資料可以寫 seed，但不要在每次啟動時無條件新增，否則資料會重複。正式 seed 要先檢查是否已存在。

### Day 9：創立 SQL Server 驗證使用者帳戶

原文重點：介紹 Windows 驗證與 SQL Server 驗證，並示範建立 SQL Server 登入帳號、設定密碼與資料庫權限。

2026 更新：安全觀念要補強。應用程式連資料庫不要用 `sa`，也不要直接給 `db_owner`。開發環境、部署帳號、應用程式執行帳號應分開。雲端環境可考慮 Microsoft Entra ID、Managed Identity 或密鑰管理服務。

練習建議：建立一個只對特定資料庫有讀寫權限的 app user，再用它連線測試。你會開始理解「權限最小化」不是口號，而是降低事故傷害範圍。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var connectionString =
    builder.Configuration.GetConnectionString("Default")
    ?? throw new InvalidOperationException("Missing connection string.");
```

完整流程：

1. 建立專用資料庫帳號。
2. 只給必要權限。
3. 將連線字串放進安全設定。
4. C# 從設定讀取。
5. 用最低權限測試 CRUD。

注意事項：連線字串可放 `appsettings.Development.json`，但真實密碼不要提交到 Git。開發機可用 User Secrets，部署環境用環境變數或 secret manager。

### Day 10：SQL 語法小精靈

原文重點：介紹 SSMS 裡的視覺化查詢設計工具，可以幫忙產生 SQL，降低初學者手寫 SQL 的壓力。

2026 更新：GUI 工具可以輔助，但不要依賴。正式工作常需要讀 log、看 ORM 產生的 SQL、分析 execution plan、手動優化查詢。只會點工具會卡在除錯階段。

練習建議：用小精靈產生一次查詢，再手動重寫一次。比較兩者差異，並確認你看得懂 `SELECT`、`FROM`、`JOIN`、`WHERE` 每一段。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
await using var command = new SqlCommand("""
    SELECT Id, Name, Country
    FROM Customers
    WHERE Country = @country;
    """, connection);

command.Parameters.Add("@country", SqlDbType.NVarChar, 50).Value = "Taiwan";
```

完整流程：

1. 先用小精靈理解查詢。
2. 手寫同等 SQL。
3. 改成參數化。
4. 在 C# 執行。
5. 用 SQL Profiler 或 EF log 檢查實際 SQL。

注意事項：SQL 小精靈產生的查詢可以學，但正式程式碼要自己看懂。參數型別與長度要盡量明確，不要所有東西都用 `AddWithValue`。

### Day 11：SQL Server 匯入 Excel 資料表

原文重點：示範把 Excel 資料匯入 SQL Server，讓外部表格資料變成資料庫資料表。

2026 更新：匯入資料時，重點不只是「匯得進去」，還要檢查欄位型別、日期格式、空值、重複資料、編碼、資料來源可信度。正式流程常會先匯入 staging table，驗證通過後再寫入正式表。

練習建議：設計一個 `ImportBatch` 表記錄匯入時間、檔名、筆數、成功/失敗狀態。這會讓資料匯入流程可追蹤。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public sealed record ImportCustomerRow(string Name, string Email);

static bool IsValid(ImportCustomerRow row)
{
    return !string.IsNullOrWhiteSpace(row.Name)
        && row.Name.Length <= 100
        && row.Email.Contains('@');
}
```

完整流程：

1. 上傳檔案。
2. 檢查大小與格式。
3. 用 parser 解析。
4. 每列做欄位驗證。
5. 寫入 staging。
6. 驗證通過後匯入正式表。

注意事項：正式 CSV 不要只用 `Split(',')`，因為欄位可能含逗號、引號或換行。建議用成熟 CSV parser，並把錯誤列記錄下來。

### Day 12：ASP.NET 與資料庫的聯繫

原文重點：介紹 Web.config 的 connection string，以及 Web Forms 如何透過 SqlDataSource 或程式碼連到 SQL Server。

2026 更新：ASP.NET Core 會把設定放在 `appsettings.json`、User Secrets、環境變數或雲端設定服務，並透過 Dependency Injection 傳入服務。Connection string 不應硬編碼在 C# 檔案裡，更不應提交真實密碼到 Git。

練習建議：

這個 JSON 範例說明：示範「Day 12：ASP.NET 與資料庫的聯繫」中的程式流程或 API 使用方式。 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。

```json
{
  "ConnectionStrings": {
    "Default": "Server=localhost;Database=DemoDb;Trusted_Connection=True;TrustServerCertificate=True"
  }
}
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
builder.Services.AddDbContext<AppDbContext>(options =>
{
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("Default"));
});
```

完整流程：

1. 在 appsettings 設定連線字串。
2. Program.cs 註冊 DbContext。
3. 透過 DI 注入。
4. 在 Page/Service 使用。
5. 不在 UI 層到處 new 連線。

注意事項：不要在 Page 或 Controller 裡手動 new `SqlConnection` 到處散落。透過 DI 管理資料存取物件，程式會更好測試與維護。

### Day 13：ASP.NET 讀取資料庫常用的 DataReader

原文重點：`SqlDataReader` 是 forward-only、read-only 的資料讀取器，適合逐筆讀取查詢結果，記憶體使用相對低。

2026 更新：DataReader 仍然有用，特別是大量讀取與高效能場景。但現代 C# 要注意 `using`、非同步 API、取消權杖與參數化查詢。不要讓連線開著不關。

練習建議：寫一個查詢客戶清單的 method，回傳 DTO，不要直接把 DataReader 綁到頁面。這會讓你的程式比較容易測試與重用。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var customers = new List<CustomerDto>();

await using var command = new SqlCommand(CustomerSql.SelectByCountry, connection);
command.Parameters.Add("@country", SqlDbType.NVarChar, 50).Value = "Taiwan";

await using var reader = await command.ExecuteReaderAsync();
while (await reader.ReadAsync())
{
    customers.Add(new CustomerDto(
        reader.GetInt32(0),
        reader.GetString(1),
        reader.IsDBNull(2) ? null : reader.GetString(2)));
}
```

完整流程：

1. 開啟連線。
2. 建立 SqlCommand。
3. 加入參數。
4. ExecuteReaderAsync。
5. 逐列轉成 DTO。
6. 關閉 reader 與 connection。

注意事項：讀欄位前要處理 NULL，否則 `GetString` 會丟例外。DataReader 適合高效讀取，但不要把它直接傳到 UI 層。

### Day 14：ASP.NET 與資料庫的資料存取：DataSet

原文重點：`DataSet` / `DataTable` 可把資料載入記憶體，像離線資料表一樣操作，再綁定到 GridView。

2026 更新：DataSet 在 Web Forms 時代很常見，但現代 Web API 或 ASP.NET Core 專案較常用 strongly typed models、DTO、EF Core entities。DataSet 仍可用於舊系統、報表、離線資料處理，但不是新專案首選。

練習建議：理解 DataSet 即可，不必把它當主要架構。新專案練習可以改用 `List<CustomerDto>` 或 EF Core 查詢結果。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public sealed record CustomerDto(int Id, string Name, string? Email);

public async Task<IReadOnlyList<CustomerDto>> GetCustomersAsync()
{
    return await db.Customers
        .OrderBy(customer => customer.Name)
        .Select(customer => new CustomerDto(
            customer.Id,
            customer.Name,
            customer.Email))
        .ToListAsync();
}
```

完整流程：

1. 定義 DTO。
2. 查詢資料。
3. Select 投影成 DTO。
4. 回傳 IReadOnlyList。
5. UI 或 API 只吃 DTO，不直接碰 DataSet。

注意事項：DataSet 很彈性，但型別不明確。新專案用 DTO 可讀性更好，也比較適合 API 與測試。

### Day 15：SQL 語法：撈取資料 SELECT

原文重點：介紹 `SELECT`、`FROM`、`WHERE`，以及使用 `*` 查全部欄位、指定欄位、條件查詢。

2026 更新：`SELECT *` 在練習可以用，正式系統請列出欄位。原因是 API contract、效能、索引覆蓋、資料外洩風險都跟欄位選擇有關。另外字串條件要注意 collation、大小寫與排序規則。

練習建議：

```sql
-- 範例用途：示範「Day 15：SQL 語法：撈取資料 SELECT」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
SELECT Id, Name, Country
FROM Customers
WHERE Country = N'Taiwan'
ORDER BY Name;
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
app.MapGet("/customers", async (AppDbContext db, string? country) =>
{
    var query = db.Customers.AsQueryable();

    if (!string.IsNullOrWhiteSpace(country))
    {
        query = query.Where(customer => customer.Country == country);
    }

    return await query.OrderBy(customer => customer.Name).ToListAsync();
});
```

完整流程：

1. 接收查詢條件。
2. 建立 IQueryable。
3. 依條件加 Where。
4. 加排序。
5. ToListAsync 執行。
6. 回傳結果。

注意事項：不要先 `ToListAsync()` 再用 C# 過濾，這會把整張表拉回記憶體。應該先組 IQueryable，最後才執行查詢。

### Day 16：表與表之間的關係 JOIN

原文重點：介紹 `INNER JOIN`、`LEFT JOIN` 等表格關聯查詢。INNER JOIN 只取兩邊都有匹配的資料；LEFT JOIN 保留左表所有資料。

2026 更新：JOIN 是 SQL 的核心。今天還應補 `RIGHT JOIN` 較少必要、`FULL OUTER JOIN` 用於找兩邊差異、以及 JOIN 條件錯誤會造成資料倍增。效能上，JOIN 欄位通常需要適合索引。

練習建議：

```sql
-- 範例用途：示範「Day 16：表與表之間的關係 JOIN」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
SELECT o.Id, c.Name, o.CreatedAt
FROM Orders AS o
INNER JOIN Customers AS c ON c.Id = o.CustomerId;
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var orders = await db.Orders
    .Include(order => order.Customer)
    .OrderByDescending(order => order.CreatedAt)
    .Select(order => new
    {
        order.Id,
        CustomerName = order.Customer!.Name,
        order.CreatedAt
    })
    .ToListAsync();
```

完整流程：

1. 確認資料表關聯。
2. 決定要 Include 還是 Select 投影。
3. 寫查詢。
4. 檢查產生 SQL。
5. 確認沒有 N+1 query。

注意事項：`Include` 適合載入關聯物件；若只要輸出欄位，`Select` 投影通常更精準。留意 N+1 query 問題。

### Day 17：集合運算 UNION、INTERSECT、EXCEPT

原文重點：介紹 SQL 的集合運算。`UNION` 合併結果並去重；`UNION ALL` 不去重；`INTERSECT` 取交集；`EXCEPT` 取差集。

2026 更新：集合運算在資料比對、報表與資料清理時很實用。要注意欄位數量、型別順序必須相容。若不需要去重，優先用 `UNION ALL`，通常效能較好。

練習建議：用 `EXCEPT` 比對「應該存在但缺少」的資料，例如訂單明細中出現但產品表不存在的 ProductId。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var studentNames = db.Students.Select(student => student.Name);
var teacherNames = db.Teachers.Select(teacher => teacher.Name);

var allNames = await studentNames
    .Union(teacherNames)
    .OrderBy(name => name)
    .ToListAsync();
```

完整流程：

1. 準備兩組同型別查詢。
2. 決定是否去重。
3. Union 或 Concat。
4. 加排序。
5. 執行並確認筆數。

注意事項：`Union` 會去重；如果你需要保留重複資料，SQL 是 `UNION ALL`，LINQ 可用 `Concat`。

### Day 18：排序 ORDER BY

原文重點：介紹 `ORDER BY` 可用 `ASC` 或 `DESC` 排序，也可以多欄位排序。

2026 更新：如果查詢結果要分頁，排序必須穩定。只用日期排序可能會有多筆同時間資料，建議再加主鍵當 tie-breaker，例如 `ORDER BY CreatedAt DESC, Id DESC`。

練習建議：

```sql
-- 範例用途：示範「Day 18：排序 ORDER BY」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
SELECT Id, Title, CreatedAt
FROM Posts
ORDER BY CreatedAt DESC, Id DESC;
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var query = db.Products.AsQueryable();

query = sortBy switch
{
    "price" => query.OrderBy(product => product.Price),
    "name" => query.OrderBy(product => product.Name),
    _ => query.OrderBy(product => product.Id)
};
```

完整流程：

1. 定義允許排序欄位白名單。
2. 用 switch 選排序。
3. 加穩定的第二排序欄位。
4. 執行查詢。
5. 防止使用者任意指定 SQL。

注意事項：不要把使用者輸入直接串進 `ORDER BY` SQL。排序欄位請用白名單 switch 控制。

### Day 19：聚合函數 COUNT、AVG、MAX、MIN、SUM

原文重點：介紹聚合函數的基本用法，例如計算筆數、平均、最大、最小與總和。

2026 更新：要注意 `COUNT(*)` 與 `COUNT(Column)` 差異：後者不計 NULL。AVG、SUM 遇到整數與 decimal 型別時也要注意精度。報表查詢常需要搭配 GROUP BY 與索引。

練習建議：

```sql
-- 範例用途：示範「Day 19：聚合函數 COUNT、AVG、MAX、MIN、SUM」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
SELECT
    COUNT(*) AS TotalRows,
    COUNT(Phone) AS RowsWithPhone,
    MAX(CreatedAt) AS LatestCreatedAt
FROM Customers;
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var summary = await db.Products
    .GroupBy(product => 1)
    .Select(group => new
    {
        Count = group.Count(),
        AveragePrice = group.Average(product => product.Price),
        MaxPrice = group.Max(product => product.Price),
        MinPrice = group.Min(product => product.Price),
        TotalPrice = group.Sum(product => product.Price)
    })
    .SingleAsync();
```

完整流程：

1. 決定統計指標。
2. 用 Count/Average/Max/Min/Sum。
3. 處理空資料。
4. 回傳 summary DTO。
5. 顯示報表。

注意事項：空集合做 `Average`、`Max`、`Min` 可能出錯。正式報表要處理沒有資料的情況。

### Day 20：GROUP BY 資料分組

原文重點：介紹 `GROUP BY` 用來把資料依某些欄位分組，再搭配聚合函數統計。

2026 更新：`WHERE` 是分組前過濾，`HAVING` 是分組後過濾。這點初學者很常混淆。現代 SQL 也常用 Window Functions 在不壓縮資料列的情況下做統計。

練習建議：

```sql
-- 範例用途：示範「Day 20：GROUP BY 資料分組」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
SELECT CustomerId, COUNT(*) AS OrderCount
FROM Orders
GROUP BY CustomerId
HAVING COUNT(*) >= 3;
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var orderCounts = await db.Orders
    .GroupBy(order => order.CustomerId)
    .Select(group => new
    {
        CustomerId = group.Key,
        Count = group.Count()
    })
    .Where(row => row.Count >= 3)
    .ToListAsync();
```

完整流程：

1. 決定分組欄位。
2. GroupBy。
3. Select 統計欄位。
4. 用 Where/Having 過濾分組結果。
5. 檢查 SQL。

注意事項：在 LINQ 中，能不能翻譯成 SQL 很重要。請用 logging 看 EF Core 產生的 SQL，避免不小心在記憶體分組。

### Day 21：IN 與 NOT IN

原文重點：介紹 `IN` 可比對多個值，`NOT IN` 則排除多個值。

2026 更新：`NOT IN` 遇到 NULL 可能產生意外結果。很多情境下，`EXISTS` / `NOT EXISTS` 更穩定、更容易表達關聯存在與否。

練習建議：

```sql
-- 範例用途：示範「Day 21：IN 與 NOT IN」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
SELECT c.Id, c.Name
FROM Customers AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders AS o
    WHERE o.CustomerId = c.Id
);
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var countries = new[] { "Taiwan", "Japan", "USA" };

var customers = await db.Customers
    .Where(customer => countries.Contains(customer.Country))
    .ToListAsync();
```

完整流程：

1. 準備允許值清單。
2. 用 Contains 產生 IN。
3. 限制清單大小。
4. 大量值改用 temp table 或 TVP。
5. 驗證 NULL 行為。

注意事項：集合太大時不要直接塞進 `IN (...)`，可能造成 SQL 過長或效能差。大量條件可考慮 temp table、table-valued parameter 或 join。

### Day 22：SQL 子查詢

原文重點：介紹在 SELECT 內再放 SELECT，用子查詢取得條件或資料。

2026 更新：子查詢、CTE、JOIN、APPLY 都可以解決類似問題。不要只背語法，要學會判斷哪一種可讀性與效能較好。SQL Server optimizer 很強，但複雜查詢仍需要看 execution plan。

練習建議：同一題用子查詢、JOIN、CTE 各寫一次。例如「查出有下過訂單的客戶」。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var customersWithOrders = await db.Customers
    .Where(customer => db.Orders.Any(order => order.CustomerId == customer.Id))
    .ToListAsync();
```

完整流程：

1. 先描述存在性條件。
2. 用 Any 寫 EXISTS。
3. 比較 JOIN 寫法。
4. 檢查 SQL。
5. 確認沒有不必要的 Count。

注意事項：`Any` 通常比 `Count() > 0` 更符合「是否存在」的語意，也常能產生較合理的 SQL。

### Day 23：INSERT 新增資料的方法

原文重點：介紹 `INSERT INTO ... VALUES`，以及從另一張表查出資料再新增。

2026 更新：正式系統新增資料要考慮預設值、NOT NULL、唯一限制、交易、錯誤處理、批次新增效能。若要取得新增後的 Id，可用 `OUTPUT INSERTED.Id` 或 ORM 回填。

練習建議：

```sql
-- 範例用途：示範「Day 23：INSERT 新增資料的方法」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
INSERT INTO Customers (Name, Country)
OUTPUT INSERTED.Id
VALUES (N'New Customer', N'Taiwan');
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var customer = new Customer
{
    Name = request.Name,
    Email = request.Email
};

db.Customers.Add(customer);
await db.SaveChangesAsync();

return Results.Created($"/customers/{customer.Id}", customer);
```

完整流程：

1. 接收新增 request。
2. 做 validation。
3. 建立 Entity。
4. Add。
5. SaveChangesAsync。
6. 回傳新增後 Id。

注意事項：不要相信 client 傳來的 Id。新增資料時通常由資料庫產生 Id，後端儲存後再回傳。

### Day 24：UPDATE 修改資料的方法

原文重點：介紹 `UPDATE ... SET ... WHERE ...`，用條件修改資料。

2026 更新：最重要提醒是一定要有 WHERE，並先 SELECT 確認會影響哪些資料。正式環境大批次 UPDATE 要考慮鎖定、交易 log、批次分段與備份。

練習建議：

```sql
-- 範例用途：示範「Day 24：UPDATE 修改資料的方法」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
SELECT *
FROM Products
WHERE Id = 10;

UPDATE Products
SET Price = 1500
WHERE Id = 10;
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var product = await db.Products.FindAsync(id);
if (product is null)
{
    return Results.NotFound();
}

product.Name = request.Name;
product.Price = request.Price;

await db.SaveChangesAsync();
return Results.NoContent();
```

完整流程：

1. 用 Id 查資料。
2. 不存在回 404。
3. 修改允許欄位。
4. SaveChangesAsync。
5. 回傳 NoContent 或更新後資料。

注意事項：先查是否存在，再更新。若多人同時修改重要資料，應補 concurrency token，例如 `rowversion`。

### Day 25：DELETE 刪除的使用方法

原文重點：介紹 `DELETE FROM ... WHERE ...`，刪除符合條件的資料。

2026 更新：很多正式系統不直接硬刪除，而是軟刪除，例如加 `DeletedAt` 或 `IsDeleted`。原因是稽核、復原、歷史資料與關聯資料都可能需要保留。若要硬刪，請搭配交易與備份。

練習建議：

```sql
-- 範例用途：示範「Day 25：DELETE 刪除的使用方法」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
UPDATE Messages
SET DeletedAt = SYSUTCDATETIME()
WHERE Id = @id;
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var message = await db.Messages.FindAsync(id);
if (message is null)
{
    return Results.NotFound();
}

message.DeletedAt = DateTimeOffset.UtcNow;
await db.SaveChangesAsync();

return Results.NoContent();
```

完整流程：

1. 查資料。
2. 檢查權限。
3. 設定 DeletedAt 或 IsDeleted。
4. SaveChangesAsync。
5. 查詢端排除軟刪資料。

注意事項：軟刪除後，所有查詢都要記得排除 `DeletedAt != null` 的資料。可用 EF Core global query filter 統一處理。

### Day 26：數學運算子與數學函數

原文重點：介紹 SQL 中的數學運算，例如加減乘除、餘數、常見數學函數。

2026 更新：數學函數本身沒過時，但要注意資料型別。整數除法、小數精度、四捨五入規則、金額欄位使用 `decimal` 而非浮點數，這些都會影響結果正確性。

練習建議：建立一張訂單明細表，練習用 SQL 算小計、折扣、稅額，但金額欄位使用 `decimal(18, 2)`。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public static decimal CalculateTotal(decimal unitPrice, int quantity, decimal discount)
{
    var subtotal = unitPrice * quantity;
    return Math.Round(subtotal - discount, 2, MidpointRounding.AwayFromZero);
}
```

完整流程：

1. 確認金額型別。
2. 用 decimal 計算。
3. 套用折扣與稅額。
4. 明確四捨五入。
5. 寫測試覆蓋邊界值。

注意事項：金額請用 `decimal`，不要用 `double`。商業規則中的四捨五入要明確定義，不能靠預設行為猜。

### Day 27：使用 ASP.NET 把檔案內的資料存進資料庫並顯示在 Web 頁面

原文重點：示範讀取檔案資料，例如 CSV，寫入 SQL Server，並顯示在 GridView。

2026 更新：檔案匯入是高風險功能。今日要補檔案大小限制、格式驗證、編碼處理、CSV parser、欄位驗證、錯誤列回報、上傳安全與批次交易。不要用簡單字串 split 當正式 CSV parser，因為 CSV 可能有逗號、引號、換行。

練習建議：匯入流程分三步：上傳檔案 -> 解析到 staging model -> 驗證通過才寫入正式表。錯誤資料要能告訴使用者第幾列錯。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
app.MapPost("/imports/customers", async (
    IFormFile file,
    AppDbContext db) =>
{
    if (file.Length == 0 || file.Length > 2 * 1024 * 1024)
    {
        return Results.BadRequest("Invalid file size.");
    }

    // 實務上請交給 CSV parser，這裡只保留流程示意。
    await using var stream = file.OpenReadStream();
    using var reader = new StreamReader(stream);
    var header = await reader.ReadLineAsync();

    return Results.Accepted();
});
```

完整流程：

1. 限制檔案。
2. 開 stream。
3. parser 逐列讀取。
4. 驗證每列。
5. 批次寫入。
6. 回報成功與失敗筆數。

注意事項：上傳檔案要限制大小與副檔名，但副檔名不可靠。正式系統還要掃描內容、驗證 MIME、記錄匯入批次與錯誤列。

### Day 28：使用 ASP.NET 與 SQL Server 做出留言板：上篇

原文重點：建立留言板資料庫與資料表，製作新增留言頁面，讓使用者輸入標題、暱稱、內容並寫入資料庫。

2026 更新：留言板是很好的 CRUD 練習，但安全不可省。新增留言要做 model validation、長度限制、HTML encoding、CSRF token、頻率限制。若允許 HTML，要用 sanitizer，不是關掉保護就好。

練習建議：用 ASP.NET Core Razor Pages 重做這篇，只做「新增留言」與「留言列表」兩個功能，先把資料流跑順。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
app.MapPost("/messages", async (
    CreateMessageRequest request,
    AppDbContext db) =>
{
    if (string.IsNullOrWhiteSpace(request.Content))
    {
        return Results.BadRequest("Content is required.");
    }

    var message = new Message
    {
        Title = request.Title,
        Content = request.Content,
        CreatedAt = DateTimeOffset.UtcNow
    };

    db.Messages.Add(message);
    await db.SaveChangesAsync();

    return Results.Created($"/messages/{message.Id}", message);
});
```

完整流程：

1. 建立留言 request。
2. 後端驗證。
3. 建立 Message entity。
4. 寫入資料庫。
5. 回傳 201。
6. 頁面導回列表。

注意事項：新增留言要限制標題與內容長度。後端驗證不能省，因為前端驗證可以被繞過。

### Day 29：留言板：中篇

原文重點：延續留言板功能，顯示留言列表與留言內容，可能包含頁面跳轉、資料繫結與詳細頁。

2026 更新：顯示使用者輸入是 XSS 最常出事的位置。Razor 預設會 HTML encode，請不要隨便用 `Html.Raw()` 顯示使用者輸入。若是 Web Forms，要確認輸出控制項的編碼行為。

練習建議：在留言內容輸入 `<script>alert(1)</script>`，確認頁面只會顯示文字，不會執行。這是最基本的 XSS 測試。

#### 完整實作流程、Coding 例子與注意事項

```cshtml
<!-- 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
@foreach (var message in Model.Messages)
{
    <article>
        <h2>@message.Title</h2>
        <p>@message.Content</p>
    </article>
}
```

完整流程：

1. 查出留言。
2. 傳給 Razor Page。
3. 使用預設編碼顯示。
4. 測試 script 字串不會執行。
5. 需要 HTML 時先 sanitizer。

注意事項：Razor 的 `@message.Content` 預設會 HTML encode。不要對使用者輸入使用 `@Html.Raw(message.Content)`，除非你已經做過可靠的 HTML sanitizer。

### Day 30：留言板：下篇

原文重點：完成留言板功能，包含回覆留言、首頁顯示、留言與回覆資料表關聯。

2026 更新：留言與回覆是典型一對多關係。今日實作應補上授權：誰可以回覆、誰可以刪除、管理員權限如何判斷。另外要補交易，避免留言與回覆狀態不一致。

練習建議：把留言板資料模型寫成：

```text
Message 1 -> many Reply
User 1 -> many Message
User 1 -> many Reply
```

然後再決定哪些頁面需要登入。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var messageExists = await db.Messages.AnyAsync(message => message.Id == request.MessageId);
if (!messageExists)
{
    return Results.NotFound("Message not found.");
}

db.Replies.Add(new Reply
{
    MessageId = request.MessageId,
    Content = request.Content,
    CreatedAt = DateTimeOffset.UtcNow
});

await db.SaveChangesAsync();
```

完整流程：

1. 確認留言存在。
2. 驗證回覆內容。
3. 建立 Reply。
4. 寫入資料庫。
5. 回到留言詳細頁。
6. 顯示最新回覆。

注意事項：新增回覆前要確認留言存在，也要確認使用者有權限回覆。不要只靠前端隱藏按鈕控制權限。

### Day 31：GridView 控制項的刪除事件刪除資料

原文重點：介紹 GridView 的 CommandField、TemplateField、RowDeleting 事件，取得 DataKey 後刪除資料，並用前端 confirm 避免誤刪。

2026 更新：前端 confirm 只是使用者體驗，不是安全控制。刪除功能一定要在後端檢查授權與 CSRF。Web Forms 事件模型適合維護舊系統；新專案請用 Razor Pages handler、MVC action 或 API endpoint。

練習建議：刪除功能要經過三層確認：UI 確認、後端授權、資料庫交易。只靠 JavaScript confirm 不夠。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
app.MapDelete("/messages/{id:int}", async (
    int id,
    ClaimsPrincipal user,
    AppDbContext db) =>
{
    var message = await db.Messages.FindAsync(id);
    if (message is null)
    {
        return Results.NotFound();
    }

    if (!user.IsInRole("Admin"))
    {
        return Results.Forbid();
    }

    message.DeletedAt = DateTimeOffset.UtcNow;
    await db.SaveChangesAsync();

    return Results.NoContent();
});
```

完整流程：

1. 接收刪除要求。
2. 後端確認登入與角色。
3. 查資料。
4. 軟刪或硬刪。
5. SaveChangesAsync。
6. 回傳結果。

注意事項：刪除是高風險操作。前端 confirm 只能防誤按，不能防惡意請求；後端一定要檢查登入、角色與 CSRF。

### Day 32：FormsAuthentication 授權驗證與會員登入

原文重點：介紹 FormsAuthentication、Ticket、Cookie，用登入後產生的驗證票來維持會員狀態。

2026 更新：FormsAuthentication 是 ASP.NET Web Forms / .NET Framework 時代的做法。新 ASP.NET Core 專案建議用 ASP.NET Core Identity、Cookie Authentication、OpenID Connect 或外部登入。密碼不可明文存放，也不建議自己設計 token 格式。

練習建議：若你學新技術，直接建立 ASP.NET Core Identity 專案，觀察登入、登出、註冊、Cookie、Claims、Roles 是怎麼串起來的。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
builder.Services
    .AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/login";
        options.AccessDeniedPath = "/access-denied";
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    });
```

完整流程：

1. 設定 Cookie Authentication。
2. 建立登入頁。
3. 驗證帳密。
4. 建立 ClaimsPrincipal。
5. SignInAsync。
6. 後續頁面用 Authorize 保護。

注意事項：這只是 Cookie authentication 設定，不包含完整會員系統。正式會員、密碼、註冊、重設密碼、MFA，建議使用 ASP.NET Core Identity 或外部 OIDC provider。

### Day 33：SQL Server 暫存表與 CTE

原文重點：介紹 `#temp table`、`##global temp table`、table variable `@table`，以及 CTE。這些都可用來處理中間查詢結果。

2026 更新：這篇仍然實用。要補的是效能與生命週期差異：`#temp` 存在 tempdb，適合較複雜中間資料；table variable 適合較小資料，但 SQL Server 新版本對估算已有改善；CTE 主要提升可讀性，不一定會把結果實體化。

練習建議：同一個報表查詢分別用 CTE 與 temp table 寫一次，觀察可讀性與 execution plan。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var sql = """
WITH RankedOrders AS (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY CustomerId
               ORDER BY CreatedAt DESC, Id DESC
           ) AS RowNo
    FROM Orders
)
SELECT Id, CustomerId, CreatedAt
FROM RankedOrders
WHERE RowNo = 1;
""";

var latestOrders = await db.LatestOrderDtos
    .FromSqlRaw(sql)
    .ToListAsync();
```

完整流程：

1. 先在 SSMS 寫好 CTE。
2. 確認結果。
3. 放進 C# raw SQL。
4. 參數化。
5. 映射 DTO。
6. 檢查 SQL injection 風險。

注意事項：`FromSqlRaw` 不可直接串使用者輸入。若有參數，使用 `FromSqlInterpolated` 或 SQL parameters。

### Day 34：ROW_NUMBER()

原文重點：介紹 `ROW_NUMBER() OVER (ORDER BY ...)` 產生排序編號，也示範 `PARTITION BY` 分組後編號。可用於分頁或取每組第一筆。

2026 更新：Window Functions 仍是 SQL 進階查詢重點。除了 `ROW_NUMBER()`，還值得學 `RANK()`、`DENSE_RANK()`、`LAG()`、`LEAD()`、`SUM() OVER()`。分頁可以用 ROW_NUMBER，也可以用 `OFFSET FETCH` 或 Keyset pagination。

練習建議：

```sql
-- 範例用途：示範「Day 34：ROW_NUMBER()」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
SELECT *
FROM (
    SELECT
        ROW_NUMBER() OVER (
            PARTITION BY CustomerId
            ORDER BY CreatedAt DESC, Id DESC
        ) AS RowNo,
        *
    FROM Orders
) AS x
WHERE x.RowNo = 1;
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var latestOrders = await db.Orders
    .GroupBy(order => order.CustomerId)
    .Select(group => group
        .OrderByDescending(order => order.CreatedAt)
        .ThenByDescending(order => order.Id)
        .First())
    .ToListAsync();
```

完整流程：

1. 定義每組排序規則。
2. 用 ROW_NUMBER 或 LINQ GroupBy。
3. 取第一筆。
4. 驗證同時間資料排序。
5. 檢查效能。

注意事項：不是所有 LINQ 都能翻譯成你期待的 SQL。請檢查 EF Core 產生的 SQL；若翻譯不佳，可以改用 raw SQL 或資料庫 view。

### Day 35：ASP.NET 使用者控制項：分頁的用法

原文重點：在 Web Forms 中建立使用者控制項，透過 limit、totalitems、targetpage 等參數產生分頁 UI，並搭配 SQL 查詢指定頁數資料。

2026 更新：這篇的「分頁概念」仍有價值，但 Web Forms 控制項實作方式已不適合作為新專案方向。現代 Web 常把分頁參數設計成 `page`、`pageSize` 或 cursor，後端回傳資料與總筆數，前端負責 UI。

練習建議：重新設計成 API 回應：

這個 JSON 範例說明：示範「Day 35：ASP.NET 使用者控制項：分頁的用法」中的程式流程或 API 使用方式。 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。

```json
{
  "items": [],
  "page": 1,
  "pageSize": 20,
  "totalItems": 135
}
```

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
app.MapGet("/messages", async (
    int page,
    int pageSize,
    AppDbContext db) =>
{
    page = Math.Max(page, 1);
    pageSize = Math.Clamp(pageSize, 1, 100);

    var totalItems = await db.Messages.CountAsync();
    var items = await db.Messages
        .OrderByDescending(message => message.CreatedAt)
        .ThenByDescending(message => message.Id)
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToListAsync();

    return Results.Ok(new { page, pageSize, totalItems, items });
});
```

完整流程：

1. 接收 page/pageSize。
2. 修正非法值。
3. Count 總筆數。
4. OrderBy/Skip/Take。
5. 回傳 items 與分頁資訊。

注意事項：分頁一定要限制 `pageSize` 最大值，否則使用者可以一次要求十萬筆。排序要穩定，避免翻頁重複或漏資料。

### Day 36：SQL Server 找出最新日期的所有資料

原文重點：要查出最新日期那一天的所有資料。原文使用 `CONVERT` 把 datetime 轉成日期格式，再用 `MAX` 找最新日期。

2026 更新：概念正確，但寫法要注意索引。若在欄位上套 `CONVERT` 或函數，可能讓索引較難有效使用。建議先找出最新日期，再用日期範圍查詢。

練習建議：

```sql
-- 範例用途：示範「Day 36：SQL Server 找出最新日期的所有資料」中的資料庫查詢或資料異動。
-- 參數說明：@ 開頭參數、WHERE 條件、表名與欄位名要依實際資料表與查詢需求調整。
-- 回傳結果 / 副作用：SELECT 會回傳結果集；INSERT / UPDATE / DELETE 會改變資料內容。
DECLARE @latestDate date = (
    SELECT MAX(CAST(CreatedAt AS date))
    FROM Orders
);

SELECT *
FROM Orders
WHERE CreatedAt >= @latestDate
  AND CreatedAt < DATEADD(day, 1, @latestDate);
```

這會比把每一列都轉成字串再比較更清楚，也更符合正式系統的查詢習慣。

#### 完整實作流程、Coding 例子與注意事項

```csharp
// 範例用途：示範「完整實作流程、Coding 例子與注意事項」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
var latestDate = await db.Orders
    .MaxAsync(order => order.CreatedAt.Date);

var nextDate = latestDate.AddDays(1);

var latestDateOrders = await db.Orders
    .Where(order => order.CreatedAt >= latestDate
        && order.CreatedAt < nextDate)
    .OrderBy(order => order.Id)
    .ToListAsync();
```

完整流程：

1. 決定最新「時間點」或最新「日期」。
2. 查出最新日期。
3. 換算日期範圍。
4. 查該範圍資料。
5. 處理 UTC 與本地時區。

注意事項：若 `CreatedAt` 是 `DateTimeOffset`，要確認時區定義。報表常以本地日期計算，但資料庫常存 UTC，兩者要轉換清楚。

## 建議的現代練習專案

把原文留言板改成 2026 版：

1. 使用 `.NET 10 LTS` 建立 ASP.NET Core Razor Pages 或 MVC 專案。
2. 使用 SQL Server 2025 Developer / Express 或 Azure SQL。
3. 使用 EF Core 建立 `Message`、`Reply`、`User` 資料表。
4. 使用 migrations 管理 schema。
5. 列表頁支援搜尋、排序、分頁。
6. 新增留言時做 model validation。
7. 顯示留言時確認 Razor 預設 HTML encoding 沒被繞過。
8. 登入使用 ASP.NET Core Identity。
9. 刪除與回覆要檢查授權。
10. 加入基本單元測試或整合測試。

## 參考資料

- 原系列 RSS：[https://ithelp.ithome.com.tw/rss/series/2276](https://ithelp.ithome.com.tw/rss/series/2276)
- Microsoft .NET Support Policy：[https://dotnet.microsoft.com/en-us/platform/support/policy](https://dotnet.microsoft.com/en-us/platform/support/policy)
- Choose between .NET and .NET Framework for server apps：[https://learn.microsoft.com/en-us/aspnet/core/conceptual-overview/choosing-the-right-dotnet](https://learn.microsoft.com/en-us/aspnet/core/conceptual-overview/choosing-the-right-dotnet)
- ASP.NET Support Policy：[https://dotnet.microsoft.com/en-us/platform/support/policy/aspnet](https://dotnet.microsoft.com/en-us/platform/support/policy/aspnet)
- Latest updates and version history for SQL Server：[https://learn.microsoft.com/en-us/troubleshoot/sql/releases/download-and-install-latest-updates](https://learn.microsoft.com/en-us/troubleshoot/sql/releases/download-and-install-latest-updates)
- Microsoft ADO.NET for SQL Server and Azure SQL Database：[https://learn.microsoft.com/sql/connect/ado-net/microsoft-ado-net-sql-server](https://learn.microsoft.com/sql/connect/ado-net/microsoft-ado-net-sql-server)
- Introduction to Microsoft.Data.SqlClient：[https://learn.microsoft.com/en-us/sql/connect/ado-net/introduction-microsoft-data-sqlclient-namespace](https://learn.microsoft.com/en-us/sql/connect/ado-net/introduction-microsoft-data-sqlclient-namespace)
- SQL Injection - SQL Server：[https://learn.microsoft.com/en-us/sql/relational-databases/security/sql-injection](https://learn.microsoft.com/en-us/sql/relational-databases/security/sql-injection)
- EF Core SQL Server Provider：[https://learn.microsoft.com/en-us/ef/core/providers/sql-server/](https://learn.microsoft.com/en-us/ef/core/providers/sql-server/)


