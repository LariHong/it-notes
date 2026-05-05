# C# Unit Test 30 天手把手教學 2026 更新整理

來源：
- iThome 鐵人賽系列頁：<https://ithelp.ithome.com.tw/2020-12th-ironman/articles/3897>
- 原系列：你就是都不寫測試才會沒時間：Kuma 的 30 天 Unit Test 手把手教學，從理論到實戰（Java 篇）
- 原作者：Kuma
- 整理日期：2026-05-05
- 轉換說明：原系列是 Java 篇，本筆記依使用需求改寫為 C# / .NET 版，並補上 2026 年的 .NET 測試實務。

## 來源提醒

來源是 30 篇 Day-by-Day 系列文。本文已建立 30 個對應 Day 章節；程式語言、框架與範例改為 C# / .NET，但概念脈絡對應原系列的單元測試、依賴控制、重構、TDD、Clean Architecture 與團隊流程。

## 2026 C# 版技術基準

| 類別 | 建議 |
| --- | --- |
| .NET | 新專案優先使用 .NET 10 LTS；.NET 8 與 .NET 9 都在 2026-11-10 結束支援，舊專案要規劃升級。 |
| 測試框架 | 本筆記用 xUnit 示範；NUnit、MSTest 也可，重點是團隊一致。 |
| Mock | 範例用 Moq；也可改用 NSubstitute 或 FakeItEasy。 |
| ASP.NET Core | Endpoint / Controller 整合測試使用 `Microsoft.AspNetCore.Mvc.Testing` 與 `WebApplicationFactory<Program>`。 |
| EF Core | Entity 用 unit test；Repository 與 query handler 優先用 integration test。 |
| CI | pull request 跑 unit tests；main branch 跑 unit + integration + 靜態檢查。 |

## 主線專案

### 專案最終會長成什麼

主線專案是 `OrderTestingLab`：一個簡化的訂單 API，包含折扣計算、付款、訂單建立、狀態轉移、查詢、Repository、API endpoint、背景通知與 CI 測試。這是 2026 補強的 C# 主線，原文沒有要求同一個專案貫穿 30 天。

### 需要的檔案地圖

| 路徑 | 責任 |
| --- | --- |
| `src/OrderTestingLab.Domain/` | Entity、Value Object、Domain Rule |
| `src/OrderTestingLab.Application/` | Service、Command Handler、Interface |
| `src/OrderTestingLab.Infrastructure/` | EF Core Repository、外部服務實作 |
| `src/OrderTestingLab.Api/` | Minimal API / Controller、DI、HTTP response |
| `tests/OrderTestingLab.UnitTests/` | Domain 與 Application 單元測試 |
| `tests/OrderTestingLab.IntegrationTests/` | API、Repository、DB integration tests |

### 30 天交付物地圖

| 天數 | 交付物 |
| --- | --- |
| Day 1-4 | 建立測試專案、第一批 xUnit 測試、測試分類規則 |
| Day 5-9 | 折扣、付款、例外、時間與非同步測試 |
| Day 10-17 | 用測試保護重構，處理重複、null、if、狀態模式 |
| Day 18-20 | 用 TDD 做 Password / Coupon / Order rule |
| Day 21-27 | Clean Architecture 分層、API、Service、Repository、CQRS |
| Day 28-30 | CI、團隊流程、測試文化與總檢查 |

### 主線端到端流程

```text
HTTP request -> Order endpoint -> PlaceOrderService -> Payment gateway
             -> Order entity -> OrderRepository -> DB
             -> response / log / notification
```

### 主線做完後檢查

- `dotnet test` 能跑過 unit 與 integration test。
- 成功下單會回傳 `201 Created`。
- 付款失敗會回傳可理解的錯誤。
- Repository test 使用接近真實 provider 的資料庫驗證 mapping。
- CI 會在 pull request 自動執行測試。

## Day 1 「科學 v.s. 數學」前言

### 這篇文章主要在講什麼
測試不是為了證明程式永遠正確，而是用可重複驗證的方式降低不確定性。

### 為什麼需要這個概念
沒有測試時，每次修改都靠人工記憶與手動點畫面，越到後期越不敢改。

### 學完這篇你應該會做到什麼
能說明單元測試的價值，並建立第一個 C# 測試專案。

### 核心重點
- 測試讓假設可驗證。
- 測試是設計回饋，不只是抓 bug。
- 第一個目標是保護重要行為，不是追求 100% coverage。

### 知識點補充

- [Visual Studio 內如何使用](CSharp_Unit_Test_30天手把手教學_2026更新整理知識點補充.md#visual-studio-內如何使用)
- [Day 1 手把手範例](CSharp_Unit_Test_30天手把手教學_2026更新整理知識點補充.md#day-1-手把手範例)

### 真實工作流程例子
- 工作任務：主管要求你修改訂單折扣規則，但不能讓既有 VIP 折扣壞掉。
- 你先判斷：先找折扣計算責任在哪一層，不直接改 Controller。
- 會動到：`DiscountCalculator.cs`、`DiscountCalculatorTests.cs`。
- 資料怎麼流：訂單金額與客戶等級進入 calculator，輸出折扣金額。
- 流程路線圖：需求 -> domain service -> unit test -> production code -> `dotnet test`。
- 工作中會寫 / 檢查的片段：
```csharp
// 範例用途：驗證 VIP 客戶折扣，讓後續改規則時有保護。
Assert.Equal(100m, calculator.Calculate(CustomerLevel.Vip, 1000m));
```
- 交付前驗證：跑 `dotnet test`；再用一般會員金額測一次避免只保護 VIP。
- 常見卡點：找不到折扣邏輯時，先從 API response 往 service / domain 追，不要先複製一份邏輯。

### 主線專案銜接
今天建立 `tests/OrderTestingLab.UnitTests`，接到主線專案的折扣計算。步驟：建立測試專案、加 reference、寫第一個 test。檢查：專案可編譯、測試會失敗、補實作後測試通過。

### 當天做完後檢查
- [ ] 有 unit test project。
- [ ] 至少一個 `[Fact]`。
- [ ] 知道測試失敗訊息在哪裡看。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| 測試 | `DiscountCalculatorTests.cs` | 描述 VIP 折扣預期 |
| 邏輯 | `DiscountCalculator.cs` | 計算折扣 |
| 指令 | `dotnet test` | 驗證結果 |

### 完整實作流程、範例與注意事項
1. 建立 `OrderTestingLab.UnitTests`。
2. 新增 `DiscountCalculatorTests`。
3. 先寫會失敗的測試。
4. 補上最小實作。
5. 執行 `dotnet test`。
```csharp
public sealed class DiscountCalculatorTests
{
    [Fact]
    public void Calculate_ReturnsVipDiscount()
    {
        var calculator = new DiscountCalculator();
        Assert.Equal(100m, calculator.Calculate(CustomerLevel.Vip, 1000m));
    }
}
```
注意事項：不要一開始就測所有細節；先保護最重要的商業行為。

### 如果結果和預期不同
若測試沒有被執行，確認測試類別是 public、方法有 `[Fact]`、測試專案有 reference。

### 負面例子 / 錯誤用法
錯誤做法：改完折扣只開瀏覽器手動測一次。問題是分支多時容易漏。修正：把每個重要折扣規則寫成測試案例。

### 小練習
新增一般會員 0% 折扣測試。

### Junior 常見誤解
誤以為測試是 QA 的事；其實單元測試是開發者用來保護自己改 code 的工具。

### 一句話總結
測試不是保證完美，而是讓修改有回饋。

## Day 2 「住手！你想搞死 QA 嗎？」單元測試是測試還是功能？

### 這篇文章主要在講什麼
單元測試是開發功能的一部分，不能全部丟給 QA 在最後補救。

### 為什麼需要這個概念
如果每個 bug 都等 QA 發現，修復成本會變高，團隊也會形成互相甩鍋。

### 學完這篇你應該會做到什麼
能把測試納入 definition of done。

### 核心重點
- 功能完成包含測試完成。
- QA 應驗證產品風險，不是替開發者做基本邏輯檢查。
- 單元測試是交付品質的一部分。

### 真實工作流程例子
- 工作任務：PM 要新增「訂單金額不可小於 1」的規則。
- 你先判斷：這是 application service 的輸入驗證，不是資料庫錯誤才擋。
- 會動到：`PlaceOrderService.cs`、`PlaceOrderServiceTests.cs`。
- 資料怎麼流：request amount -> service validation -> exception / order created。
- 流程路線圖：PM 需求 -> service test -> validation -> API 錯誤映射。
- 工作中會寫 / 檢查的片段：
```csharp
await Assert.ThrowsAsync<ArgumentOutOfRangeException>(
    () => service.PlaceOrderAsync(new("book", 0m), CancellationToken.None));
```
- 交付前驗證：測 0 元失敗；測 1 元成功。
- 常見卡點：只在前端擋金額，後端 API 仍可被直接呼叫。

### 主線專案銜接
今天把「金額驗證」接進下單流程。步驟：補失敗測試、補 service guard、確認 API 層可轉成 400。檢查：0 元失敗、1 元成功、錯誤訊息可理解。

### 當天做完後檢查
- [ ] service 有錯誤路徑測試。
- [ ] 測試名稱描述情境。
- [ ] definition of done 包含測試。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| DTO | `PlaceOrderCommand` | 承接輸入 |
| Service | `PlaceOrderService` | 驗證規則 |
| Test | `PlaceOrderServiceTests` | 保護錯誤路徑 |

### 完整實作流程、範例與注意事項
1. 寫 `Amount <= 0` 的測試。
2. 在 service 入口加入 guard clause。
3. 補成功案例，避免只測錯誤。
```csharp
if (command.Amount <= 0)
{
    throw new ArgumentOutOfRangeException(nameof(command.Amount));
}
```
注意事項：驗證規則不要散在 Controller、Service、Repository 三份。

### 如果結果和預期不同
若測試丟出不同例外，確認 guard clause 在呼叫付款之前。

### 負面例子 / 錯誤用法
錯誤做法：等 DB constraint 擋掉 0 元訂單。問題是錯誤太晚、訊息難讀。修正：在 use case 邊界先驗證。

### 小練習
新增「商品名稱不可空白」測試。

### Junior 常見誤解
誤以為只有 UI 驗證就夠；後端仍必須保護自己的 use case。

### 一句話總結
功能沒測完，就還沒真的完成。

## Day 3 「要開始囉！」單元測試的起手式

### 這篇文章主要在講什麼
用最小範例學會 Arrange、Act、Assert。

### 為什麼需要這個概念
測試如果沒有結構，幾週後就看不懂自己在測什麼。

### 學完這篇你應該會做到什麼
能寫出清楚命名且可讀的 xUnit 測試。

### 核心重點
- Arrange 建資料。
- Act 呼叫被測行為。
- Assert 驗證結果。

### 真實工作流程例子
- 工作任務：新增「滿 1000 免運」規則。
- 你先判斷：這是運費計算邏輯，先獨立成 `ShippingFeeCalculator`。
- 會動到：`ShippingFeeCalculator.cs`、`ShippingFeeCalculatorTests.cs`。
- 資料怎麼流：訂單金額進入 calculator，輸出運費。
- 流程路線圖：需求 -> calculator test -> calculator -> checkout service。
- 工作中會寫 / 檢查的片段：
```csharp
var fee = calculator.Calculate(1000m);
Assert.Equal(0m, fee);
```
- 交付前驗證：測 999 與 1000；確認邊界。
- 常見卡點：只測 1000，忘記 999 這種邊界。

### 主線專案銜接
今天新增 `ShippingFeeCalculator`。步驟：建立 class、寫 999/1000 兩個測試、接到 checkout。檢查：邊界通過、命名清楚、測試分段。

### 當天做完後檢查
- [ ] 測試分 Arrange / Act / Assert。
- [ ] 邊界值有測。
- [ ] 測試失敗訊息清楚。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Calculator | `ShippingFeeCalculator` | 運費規則 |
| Test | `ShippingFeeCalculatorTests` | 驗證邊界 |
| 使用端 | `CheckoutService` | 呼叫運費計算 |

### 完整實作流程、範例與注意事項
1. 先寫滿 1000 免運測試。
2. 再寫 999 仍收運費測試。
3. 實作 calculator。
```csharp
public decimal Calculate(decimal amount) => amount >= 1000m ? 0m : 80m;
```
注意事項：測試名稱不要叫 `TestCalculate`，要描述情境。

### 如果結果和預期不同
若 1000 還收運費，檢查比較運算是否寫成 `>` 而不是 `>=`。

### 負面例子 / 錯誤用法
錯誤做法：只測 1200。問題是邊界 1000 可能壞。修正：補 999、1000、1001。

### 小練習
新增「負數金額丟例外」測試。

### Junior 常見誤解
誤以為測試越長越完整；其實越短越能指出問題。

### 一句話總結
清楚的測試從清楚的三段式開始。

## Day 4 「樹頭顧乎哉」Unit Test v.s. Integration Test

### 這篇文章主要在講什麼
不同層級的測試負責不同風險，不能全部混在一起。

### 為什麼需要這個概念
把資料庫、HTTP、外部 API 都塞進單元測試，會讓測試慢又不穩。

### 學完這篇你應該會做到什麼
能判斷一個測試該放 unit test 還是 integration test。

### 核心重點
- Unit test 測單一邏輯。
- Integration test 測多元件協作。
- E2E test 測使用者路徑。

### 真實工作流程例子
- 工作任務：確認 `/orders` endpoint 能新增訂單。
- 你先判斷：service 規則用 unit test；HTTP + DI + DB 用 integration test。
- 會動到：`OrderEndpointTests.cs`、`PlaceOrderServiceTests.cs`。
- 資料怎麼流：HTTP body -> endpoint -> service -> repository -> DB。
- 流程路線圖：API request -> endpoint -> service -> db -> response。
- 工作中會寫 / 檢查的片段：
```csharp
var response = await client.PostAsJsonAsync("/orders", new { itemName = "book", amount = 500 });
Assert.Equal(HttpStatusCode.Created, response.StatusCode);
```
- 交付前驗證：unit test 跑快；integration test 可驗證 status code 與 DB。
- 常見卡點：把 integration test 放在 unit 專案，導致本機跑測試變慢。

### 主線專案銜接
今天拆出 `UnitTests` 與 `IntegrationTests`。步驟：建立兩個測試專案、分類命名、CI 分階段跑。檢查：unit 不啟動 web host、integration 可建立 client、兩者都可獨立執行。

### 當天做完後檢查
- [ ] 測試專案已分層。
- [ ] Repository 不用假 unit test 硬測。
- [ ] CI 能分開跑。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Unit | `tests/...UnitTests` | 純邏輯 |
| Integration | `tests/...IntegrationTests` | API / DB |
| API | `Program.cs` | endpoint 註冊 |

### 完整實作流程、範例與注意事項
1. 建立 integration test project。
2. 加 `Microsoft.AspNetCore.Mvc.Testing`。
3. 用 `WebApplicationFactory<Program>` 建立 client。
```csharp
public sealed class OrderApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    public OrderApiTests(WebApplicationFactory<Program> factory) =>
        Client = factory.CreateClient();
    private HttpClient Client { get; }
}
```
注意事項：integration test 慢是正常的，但不要讓它取代 unit test。

### 如果結果和預期不同
若找不到 `Program`，在 API 專案加 `public partial class Program { }`。

### 負面例子 / 錯誤用法
錯誤做法：mock 掉整個 `HttpClient` 來假裝測 API。問題是路由與 DI 沒測到。修正：用 `WebApplicationFactory`。

### 小練習
把現有一個 service test 與一個 endpoint test 分到不同專案。

### Junior 常見誤解
誤以為慢測試比較完整；其實不同測試層級要互補。

### 一句話總結
測試層級清楚，測試才會又快又有保護力。

## Day 5 「乖，聽話給你吃糖果！」用資料控制依賴

### 這篇文章主要在講什麼
很多測試先用資料組合就能完成，不必一開始就 mock。

### 為什麼需要這個概念
過度 mock 會讓測試綁實作細節，反而難維護。

### 學完這篇你應該會做到什麼
能用測試資料覆蓋重要分支。

### 核心重點
- 先判斷能否用 plain object。
- 邊界資料比工具重要。
- 測試資料要清楚表達意圖。

### 真實工作流程例子
- 工作任務：折扣規則依客戶等級不同。
- 你先判斷：客戶等級是輸入資料，不需要 mock customer。
- 會動到：`Customer.cs`、`DiscountPolicyTests.cs`。
- 資料怎麼流：customer level + amount -> discount policy -> discount。
- 流程路線圖：測試資料 -> policy -> assert。
- 工作中會寫 / 檢查的片段：
```csharp
[Theory]
[InlineData(CustomerLevel.Normal, 1000, 0)]
[InlineData(CustomerLevel.Vip, 1000, 100)]
public void Calculate_ReturnsExpectedDiscount(CustomerLevel level, decimal amount, decimal expected) { }
```
- 交付前驗證：每個等級至少一筆；邊界金額至少一筆。
- 常見卡點：為每個 DTO 都建立 mock，造成測試又長又脆弱。

### 主線專案銜接
今天建立折扣資料案例。步驟：用 `[Theory]`、列出等級、補 edge case。檢查：測試資料可讀、沒有不必要 mock、失敗時知道是哪筆資料。

### 當天做完後檢查
- [ ] 用 `[Theory]` 覆蓋多筆資料。
- [ ] 測試資料命名清楚。
- [ ] 沒有 mock 純資料物件。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Data | `[InlineData]` | 測試分支 |
| Policy | `DiscountPolicy` | 商業規則 |
| Test | `DiscountPolicyTests` | 驗證輸出 |

### 完整實作流程、範例與注意事項
1. 找出規則表。
2. 把每條規則變成一筆 test data。
3. 用 `[Theory]` 執行。
```csharp
[Theory]
[InlineData(CustomerLevel.Vip, 1000, 100)]
public void Calculate_ReturnsExpectedDiscount(CustomerLevel level, decimal amount, decimal expected)
{
    Assert.Equal(expected, new DiscountPolicy().Calculate(level, amount));
}
```
注意事項：資料太多時改用 `MemberData`，不要讓 `[InlineData]` 難讀。

### 如果結果和預期不同
若 decimal 比對失敗，確認金額是否有四捨五入規則。

### 負面例子 / 錯誤用法
錯誤做法：mock `Customer.Level`。問題是 mock 掩蓋資料模型。修正：直接 new customer 或傳 enum。

### 小練習
把免運規則改成 `[Theory]` 測 999、1000、1001。

### Junior 常見誤解
誤以為有依賴就要 mock；資料物件通常直接建立即可。

### 一句話總結
能用資料說清楚的測試，就先別急著 mock。

## Day 6 「不聽話就換掉」用 Mock 工具控制依賴

### 這篇文章主要在講什麼
當依賴不可控、很慢或有副作用時，用 mock 替換它。

### 為什麼需要這個概念
單元測試不應真的扣款、寄信或打外部 API。

### 學完這篇你應該會做到什麼
能用 Moq 控制外部服務回應。

### 核心重點
- mock 外部邊界。
- 驗證輸出與必要互動。
- 不要驗證太多內部呼叫順序。

### 真實工作流程例子
- 工作任務：付款成功後才建立訂單。
- 你先判斷：付款 gateway 是外部依賴，要 mock。
- 會動到：`IPaymentGateway`、`PlaceOrderServiceTests`。
- 資料怎麼流：service -> gateway mock -> repository mock。
- 流程路線圖：command -> service -> payment -> save order。
- 工作中會寫 / 檢查的片段：
```csharp
paymentGateway.Setup(x => x.PayAsync(500m, It.IsAny<CancellationToken>())).ReturnsAsync(true);
```
- 交付前驗證：付款成功會 save；付款失敗不 save。
- 常見卡點：忘記 setup async 回傳，導致測試拿到預設值。

### 主線專案銜接
今天接上付款介面。步驟：定義 interface、注入 service、用 mock 控制成功失敗。檢查：成功 save 一次、失敗 save 零次、沒有打真 API。

### 當天做完後檢查
- [ ] 外部依賴已抽 interface。
- [ ] 成功與失敗都有測。
- [ ] 沒有真扣款。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Interface | `IPaymentGateway` | 外部付款邊界 |
| Mock | `PlaceOrderServiceTests` | 控制付款結果 |
| Service | `PlaceOrderService` | 決定是否存訂單 |

### 完整實作流程、範例與注意事項
1. 抽出 `IPaymentGateway`。
2. 在 service constructor 注入。
3. 測試中用 Moq setup。
```csharp
orderRepository.Verify(x => x.SaveAsync(It.IsAny<Order>(), It.IsAny<CancellationToken>()), Times.Once);
```
注意事項：不要 mock 被測的 `PlaceOrderService` 自己。

### 如果結果和預期不同
若 `Verify` 失敗，先確認 service 是否真的呼叫 mock object 而不是 new 了真實 gateway。

### 負面例子 / 錯誤用法
錯誤做法：service 內部 `new PaymentGateway()`。問題是測試無法替換。修正：依賴注入 interface。

### 小練習
新增付款失敗時丟 `InvalidOperationException` 的測試。

### Junior 常見誤解
誤以為 mock 越多越專業；mock 應該只放在邊界。

### 一句話總結
Mock 是用來隔離不可控依賴，不是用來複製實作。

## Day 7 「Tell. Don't Ask.」測行為

### 這篇文章主要在講什麼
有些方法沒有回傳值，仍然可以測它是否完成該做的行為。

### 為什麼需要這個概念
命令型流程常透過副作用完成工作，例如存檔、發事件、寄信。

### 學完這篇你應該會做到什麼
能測 command handler 的副作用。

### 核心重點
- Query 看回傳。
- Command 看副作用。
- 驗證協作者互動時要聚焦重要行為。

### 真實工作流程例子
- 工作任務：訂單建立後發布 `OrderPlaced` 事件。
- 你先判斷：這是 service 完成後的副作用，要驗證 event publisher。
- 會動到：`IEventPublisher`、`PlaceOrderService`、測試。
- 資料怎麼流：order saved -> event created -> publisher。
- 流程路線圖：command -> service -> repository -> publisher。
- 工作中會寫 / 檢查的片段：
```csharp
publisher.Verify(x => x.PublishAsync(It.IsAny<OrderPlaced>(), It.IsAny<CancellationToken>()), Times.Once);
```
- 交付前驗證：成功下單發布事件；付款失敗不發布。
- 常見卡點：驗證太多內部方法，導致重構就壞。

### 主線專案銜接
今天加入事件發布。步驟：定義 event、注入 publisher、測成功/失敗副作用。檢查：成功發一次、失敗不發、event 包含 order id。

### 當天做完後檢查
- [ ] command 行為有測。
- [ ] 副作用有驗證。
- [ ] 不測 private method。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Event | `OrderPlaced` | 訂單事件 |
| Publisher | `IEventPublisher` | 發布事件 |
| Test | `PlaceOrderServiceTests` | 驗證發布 |

### 完整實作流程、範例與注意事項
1. 建立 `OrderPlaced` record。
2. service 成功後呼叫 publisher。
3. 測試驗證 publisher。
```csharp
public sealed record OrderPlaced(Guid OrderId);
```
注意事項：不要驗證所有呼叫順序，除非順序本身是規則。

### 如果結果和預期不同
若 event 沒發，確認 service 是否在 save 成功後才 publish。

### 負面例子 / 錯誤用法
錯誤做法：只確認方法沒丟例外。問題是副作用可能沒做。修正：驗證關鍵協作者被呼叫。

### 小練習
新增「取消訂單後發布 `OrderCancelled`」測試。

### Junior 常見誤解
誤以為沒有 return 就不能測；副作用也是可觀察行為。

### 一句話總結
Command 測的是系統做了什麼，不只是回傳了什麼。

## Day 8 「說好的射後不理呢？」多線程環境下的單元測試

### 這篇文章主要在講什麼
非同步與並行測試要避免時間與共享狀態造成不穩。

### 為什麼需要這個概念
用 `Task.Delay` 或共享 static 狀態會讓測試偶爾失敗，最難排查。

### 學完這篇你應該會做到什麼
能寫基本 async 測試，並知道何時抽象時間。

### 核心重點
- async test 要 `await`。
- 避免 sleep-based test。
- 時間依賴用 `TimeProvider` 或 wrapper。

### 真實工作流程例子
- 工作任務：訂單 30 分鐘未付款自動過期。
- 你先判斷：時間是依賴，要可控制。
- 會動到：`TimeProvider`、`OrderExpirationService`、測試。
- 資料怎麼流：現在時間 -> expiration service -> order status。
- 流程路線圖：fake time -> service -> entity -> assert status。
- 工作中會寫 / 檢查的片段：
```csharp
var now = timeProvider.GetUtcNow();
```
- 交付前驗證：29 分鐘不過期；30 分鐘過期。
- 常見卡點：用真實現在時間導致測試跨分鐘失敗。

### 主線專案銜接
今天加入訂單過期規則。步驟：注入時間來源、寫邊界測試、避免 `Task.Delay`。檢查：29/30 分鐘結果正確、測試快速、可重複。

### 當天做完後檢查
- [ ] async 測試有 await。
- [ ] 沒有 `Thread.Sleep`。
- [ ] 時間可控制。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Time | `TimeProvider` | 提供現在時間 |
| Service | `OrderExpirationService` | 判斷過期 |
| Test | `OrderExpirationServiceTests` | 驗證邊界 |

### 完整實作流程、範例與注意事項
1. service constructor 接收 `TimeProvider`。
2. 測試提供固定時間。
3. 驗證狀態轉換。
```csharp
public bool IsExpired(DateTimeOffset createdAt) =>
    _timeProvider.GetUtcNow() - createdAt >= TimeSpan.FromMinutes(30);
```
注意事項：不要依賴測試執行當下的真實時間。

### 如果結果和預期不同
若測試偶發失敗，優先找真實時間、共享狀態、平行執行。

### 負面例子 / 錯誤用法
錯誤做法：`await Task.Delay(TimeSpan.FromMinutes(30))`。問題是測試不可接受。修正：抽象時間。

### 小練習
寫「優惠券 7 天後失效」測試。

### Junior 常見誤解
誤以為 async 測試只要回傳 `void`；xUnit async test 應回傳 `Task`。

### 一句話總結
可控制的時間，才有可相信的測試。

## Day 9 「世事難預料」單元測試與例外處理

### 這篇文章主要在講什麼
錯誤路徑也是系統合約，應該被測試保護。

### 為什麼需要這個概念
例外處理沒測，API 很容易回傳 500 或吞掉重要錯誤。

### 學完這篇你應該會做到什麼
能用 `Assert.ThrowsAsync` 測錯誤路徑。

### 核心重點
- 明確定義何時丟例外。
- API 層要把例外轉成合理 response。
- 不要吞例外。

### 真實工作流程例子
- 工作任務：付款失敗時不可建立訂單。
- 你先判斷：付款失敗是 service 錯誤路徑。
- 會動到：`PlaceOrderService`、`OrderEndpoint`、測試。
- 資料怎麼流：gateway false -> service exception -> API 400/409。
- 流程路線圖：command -> payment fail -> exception -> error response。
- 工作中會寫 / 檢查的片段：
```csharp
await Assert.ThrowsAsync<InvalidOperationException>(() => service.PlaceOrderAsync(command, ct));
```
- 交付前驗證：付款失敗不 save；API 回合理 status code。
- 常見卡點：catch 後只寫 log 沒有回報失敗。

### 主線專案銜接
今天補付款失敗路徑。步驟：setup gateway false、assert exception、verify repository never。檢查：不存訂單、不發事件、API 錯誤可讀。

### 當天做完後檢查
- [ ] 有錯誤路徑測試。
- [ ] 有 `Times.Never`。
- [ ] API 錯誤不是裸 500。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Gateway | mock | 模擬付款失敗 |
| Service | `PlaceOrderService` | 停止流程 |
| API | endpoint | 轉錯誤 response |

### 完整實作流程、範例與注意事項
1. mock 付款失敗。
2. assert 例外。
3. 驗證沒有 save。
```csharp
repository.Verify(x => x.SaveAsync(It.IsAny<Order>(), It.IsAny<CancellationToken>()), Times.Never);
```
注意事項：例外型別要有語意，不要全部丟 `Exception`。

### 如果結果和預期不同
若 repository 仍被呼叫，確認 service 是否在付款結果後立刻 return / throw。

### 負面例子 / 錯誤用法
錯誤做法：`catch { return; }`。問題是呼叫端以為成功。修正：回傳 Result 或丟明確例外。

### 小練習
新增庫存不足錯誤路徑。

### Junior 常見誤解
誤以為只要 happy path 通過就好；錯誤路徑通常更接近真實事故。

### 一句話總結
錯誤路徑不測，就等線上使用者幫你測。

## Day 10 「如入鮑魚之肆」從測試聞出重複

### 這篇文章主要在講什麼
測試會暴露 production code 與 test setup 的重複。

### 為什麼需要這個概念
重複會讓規則改動時需要改很多地方，漏改就出 bug。

### 學完這篇你應該會做到什麼
能辨識測試 setup 重複並抽成 helper / builder。

### 核心重點
- 測試重複也是維護成本。
- 抽象前先確認重複真的有共同語意。
- builder 要讓測試更可讀。

### 真實工作流程例子
- 工作任務：多個測試都建立 VIP customer。
- 你先判斷：重複的是測試資料建構，不是規則本身。
- 會動到：`CustomerBuilder.cs`、相關 tests。
- 資料怎麼流：builder -> customer -> service。
- 流程路線圖：test setup -> builder -> object -> assert。
- 工作中會寫 / 檢查的片段：
```csharp
var customer = CustomerBuilder.AVip().Build();
```
- 交付前驗證：測試仍可讀；失敗訊息仍指向情境。
- 常見卡點：helper 抽太深，測試看不出資料內容。

### 主線專案銜接
今天整理測試資料建構。步驟：找重複 setup、抽 builder、保留關鍵差異在測試內。檢查：測試更短、語意更清楚、沒有隱藏重要輸入。

### 當天做完後檢查
- [ ] 重複 setup 已整理。
- [ ] 測試仍讀得懂。
- [ ] 沒有過度抽象。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Builder | `CustomerBuilder` | 建測試資料 |
| Tests | 多個 test class | 使用語意化資料 |
| Domain | `Customer` | 被建立的物件 |

### 完整實作流程、範例與注意事項
1. 找三段以上相同 setup。
2. 抽 builder。
3. 保留測試特有資料。
```csharp
public static CustomerBuilder AVip() => new CustomerBuilder().WithLevel(CustomerLevel.Vip);
```
注意事項：builder 不要塞商業規則，只負責資料建立。

### 如果結果和預期不同
若抽完後測試變難懂，把關鍵資料顯式寫回測試。

### 負面例子 / 錯誤用法
錯誤做法：所有測試共用一個神奇 `CreateDefaultOrder()`。問題是讀者不知道預設值。修正：用具名 builder 方法。

### 小練習
為 `Order` 建一個 `OrderBuilder`。

### Junior 常見誤解
誤以為測試不能重構；測試也需要維護，但要避免過度設計。

### 一句話總結
好的測試資料工具會讓情境更清楚，而不是更神秘。

## Day 11 「我以火力掩護你」在測試保護下消除重複

### 這篇文章主要在講什麼
重構前先建立測試保護，讓你敢動 production code。

### 為什麼需要這個概念
沒有測試的重構很容易變成「順手改行為」。

### 學完這篇你應該會做到什麼
能先補 characterization test，再重構重複邏輯。

### 核心重點
- 先固定現有行為。
- 小步重構。
- 每步跑測試。

### 真實工作流程例子
- 工作任務：Controller 與 Service 都有金額驗證。
- 你先判斷：驗證規則應集中在 use case 或 validation component。
- 會動到：`PlaceOrderValidator`、Controller、Service tests。
- 資料怎麼流：request -> validator -> service。
- 流程路線圖：補測試 -> 抽 validator -> 替換重複 -> 跑測試。
- 工作中會寫 / 檢查的片段：
```csharp
var result = validator.Validate(new PlaceOrderCommand("book", 0m));
Assert.False(result.IsValid);
```
- 交付前驗證：Controller 與 Service 行為不變；重複消失。
- 常見卡點：重構時同時改規格，導致測試無法判斷是重構還是功能變更。

### 主線專案銜接
今天抽出 `PlaceOrderValidator`。步驟：補現有行為測試、抽類別、替換呼叫。檢查：舊測試通過、重複消失、錯誤訊息一致。

### 當天做完後檢查
- [ ] 重構前有測試。
- [ ] 每一步都有跑測試。
- [ ] 沒有同時改新需求。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Validator | `PlaceOrderValidator` | 集中驗證 |
| API | endpoint | 呼叫驗證 |
| Tests | validator / endpoint tests | 保護行為 |

### 完整實作流程、範例與注意事項
1. 補 Controller / Service 既有行為測試。
2. 建立 validator。
3. 替換重複邏輯。
```csharp
public ValidationResult Validate(PlaceOrderCommand command) =>
    command.Amount <= 0 ? ValidationResult.Fail("Amount must be positive.") : ValidationResult.Success();
```
注意事項：重構 commit 與功能 commit 分開會比較好 review。

### 如果結果和預期不同
若測試壞很多，先 rollback 到最小步驟，只替換一個呼叫點。

### 負面例子 / 錯誤用法
錯誤做法：一邊抽 validator 一邊改錯誤訊息與規則。問題是 review 無法判斷風險。修正：先純重構。

### 小練習
把商品名稱驗證也搬到 validator。

### Junior 常見誤解
誤以為重構就是大改；真正安全的重構通常是小步。

### 一句話總結
測試就是重構時的掩護火力。

## Day 12 「可惡想要」Feature Envy

### 這篇文章主要在講什麼
如果一個類別一直拆別人的資料做判斷，行為可能放錯地方。

### 為什麼需要這個概念
行為放錯位置會讓測試 setup 很大，需求變更也難找責任。

### 學完這篇你應該會做到什麼
能把靠近資料的行為搬回 domain object。

### 核心重點
- 資料與行為應靠近。
- Service 不應變成所有規則的垃圾桶。
- 測試困難常反映設計問題。

### 真實工作流程例子
- 工作任務：判斷訂單是否可取消。
- 你先判斷：取消規則跟 order 狀態強相關，應放在 `Order`。
- 會動到：`Order.cs`、`OrderTests.cs`、`CancelOrderService.cs`。
- 資料怎麼流：service 找 order -> order 判斷 -> service 儲存。
- 流程路線圖：command -> service -> order.CanCancel -> repository。
- 工作中會寫 / 檢查的片段：
```csharp
Assert.True(order.CanCancel(now));
```
- 交付前驗證：未付款可取消；已出貨不可取消。
- 常見卡點：所有狀態判斷都塞在 service。

### 主線專案銜接
今天把取消規則搬回 `Order`。步驟：先寫 `OrderTests`、搬移規則、service 改呼叫。檢查：service 變薄、Order 測試清楚、取消流程不變。

### 當天做完後檢查
- [ ] domain 行為有自己的測試。
- [ ] service 不再拆太多 order 欄位。
- [ ] 規則位置合理。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Entity | `Order` | 判斷取消規則 |
| Service | `CancelOrderService` | 編排流程 |
| Test | `OrderTests` | 驗證 domain rule |

### 完整實作流程、範例與注意事項
1. 在 `OrderTests` 寫取消規則。
2. 在 `Order` 加 `CanCancel`。
3. Service 改用 `order.Cancel()`。
```csharp
public bool CanCancel() => Status is OrderStatus.Pending or OrderStatus.Paid;
```
注意事項：不是所有邏輯都要進 entity；跨外部服務流程仍放 application service。

### 如果結果和預期不同
若 service 測試需要大改，先保留 service 測試，再補 domain test。

### 負面例子 / 錯誤用法
錯誤做法：`if (order.Status == ... && order.PaidAt == ...)` 散在多個 service。問題是規則重複。修正：集中到 order 方法。

### 小練習
把「是否可付款」移到 `Order.CanPay()`。

### Junior 常見誤解
誤以為 entity 只能有 property；domain entity 可以有行為。

### 一句話總結
行為越靠近資料，測試和維護越自然。

## Day 13 「難兄難弟」Data Clump 與 Primitive Obsession

### 這篇文章主要在講什麼
經常一起出現的 primitive 值，可能應該變成 value object。

### 為什麼需要這個概念
到處傳 `string email`、`decimal amount` 容易漏驗證、傳錯欄位。

### 學完這篇你應該會做到什麼
能為 Email、Money 建立 C# value object 並測試。

### 核心重點
- primitive 不代表語意清楚。
- value object 封裝驗證。
- 測試 value object 的不變條件。

### 真實工作流程例子
- 工作任務：Email 格式常在不同地方重複驗證。
- 你先判斷：Email 是 domain value object。
- 會動到：`Email.cs`、`EmailTests.cs`、Customer。
- 資料怎麼流：輸入 string -> Email.Create -> Customer。
- 流程路線圖：request email -> value object -> entity。
- 工作中會寫 / 檢查的片段：
```csharp
Assert.Throws<ArgumentException>(() => Email.Create("not-email"));
```
- 交付前驗證：合法 email 成功；不合法 email 失敗。
- 常見卡點：只靠 UI regex，後端仍接受髒資料。

### 主線專案銜接
今天新增 `Email` value object。步驟：寫合法/非法測試、建立 factory、替換 Customer email。檢查：非法值進不去 domain、測試集中、呼叫端語意清楚。

### 當天做完後檢查
- [ ] primitive 被語意化。
- [ ] value object 有測試。
- [ ] validation 不再散落。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Value Object | `Email` | 封裝 email 規則 |
| Tests | `EmailTests` | 驗證格式 |
| Entity | `Customer` | 使用 Email |

### 完整實作流程、範例與注意事項
1. 找重複 primitive。
2. 建 value object。
3. 用測試保護合法與非法值。
```csharp
public sealed record Email
{
    public string Value { get; }
    private Email(string value) => Value = value;
    public static Email Create(string value) =>
        value.Contains('@') ? new Email(value) : throw new ArgumentException("Invalid email.");
}
```
注意事項：範例很簡化；真實 email 驗證要依產品需求決定。

### 如果結果和預期不同
若序列化或 EF mapping 壞掉，補 mapping / converter integration test。

### 負面例子 / 錯誤用法
錯誤做法：所有 method 都收 `string email`。問題是任何字串都能混入。修正：改收 `Email`。

### 小練習
建立 `Money` value object，禁止負數。

### Junior 常見誤解
誤以為 value object 只是包一層；真正價值是語意與不變條件。

### 一句話總結
把重要概念從 primitive 升級成型別，錯誤會少很多。

## Day 14 「不殘而廢」Data Class

### 這篇文章主要在講什麼
只有資料沒有行為的 class 可能讓邏輯散落到外面。

### 為什麼需要這個概念
當每個 service 都讀寫同一批 property，規則會分散而難測。

### 學完這篇你應該會做到什麼
能判斷 record / DTO / entity 的責任差異。

### 核心重點
- DTO 可以只有資料。
- Domain entity 應承載規則。
- 不要把所有 class 都做成貧血模型。

### 真實工作流程例子
- 工作任務：訂單狀態轉移規則常被寫錯。
- 你先判斷：狀態轉移是 `Order` 行為。
- 會動到：`Order.Pay()`、`OrderTests.cs`。
- 資料怎麼流：service 呼叫 order.Pay -> status 改變。
- 流程路線圖：payment success -> order.Pay -> repository save。
- 工作中會寫 / 檢查的片段：
```csharp
order.Pay();
Assert.Equal(OrderStatus.Paid, order.Status);
```
- 交付前驗證：Pending 可付款；Shipped 不可付款。
- 常見卡點：直接 `order.Status = Paid` 跳過規則。

### 主線專案銜接
今天封裝訂單狀態轉移。步驟：把 setter 收起來、加方法、補狀態測試。檢查：外部不能亂改狀態、非法轉移會失敗、service 更清楚。

### 當天做完後檢查
- [ ] Entity 有行為。
- [ ] 狀態 setter 不被外部亂改。
- [ ] DTO 與 Entity 分清楚。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Entity | `Order` | 管狀態 |
| DTO | `PlaceOrderRequest` | 承接 API |
| Test | `OrderTests` | 驗證轉移 |

### 完整實作流程、範例與注意事項
1. 將 `Status` 改成 private set。
2. 新增 `Pay()`。
3. 測合法與非法轉移。
```csharp
public void Pay()
{
    if (Status != OrderStatus.Pending) throw new InvalidOperationException("Only pending orders can be paid.");
    Status = OrderStatus.Paid;
}
```
注意事項：DTO 沒行為是正常的；不要把 DTO 當 domain entity。

### 如果結果和預期不同
若 EF Core 需要 setter，使用 private setter 或 backing field mapping。

### 負面例子 / 錯誤用法
錯誤做法：`order.Status = OrderStatus.Paid` 到處出現。問題是規則被繞過。修正：只允許 `order.Pay()`。

### 小練習
新增 `Ship()` 並限制只有 Paid 可出貨。

### Junior 常見誤解
誤以為 C# record 就一定是 domain model；record 常適合 DTO / value object，但仍要看責任。

### 一句話總結
重要狀態不要裸奔，讓物件自己保護規則。

## Day 15 「一切皆空」Null

### 這篇文章主要在講什麼
Null 是常見錯誤來源，應用明確策略處理。

### 為什麼需要這個概念
Null 沒有語意，線上最常見的 crash 之一就是未處理 null。

### 學完這篇你應該會做到什麼
能用 nullable reference types、guard clause 或 Result 讓 null 行為明確。

### 核心重點
- 啟用 nullable。
- 邊界入口檢查 null。
- 不要把 null 當隱藏狀態。

### 真實工作流程例子
- 工作任務：商品名稱 null 時 API 會 500。
- 你先判斷：request validation 應在入口或 use case 邊界。
- 會動到：`PlaceOrderRequest`、validator、endpoint test。
- 資料怎麼流：JSON null -> request -> validator -> 400。
- 流程路線圖：HTTP body -> model binding -> validation -> response。
- 工作中會寫 / 檢查的片段：
```csharp
Assert.False(validator.Validate(new PlaceOrderCommand(null!, 100m)).IsValid);
```
- 交付前驗證：null、空字串、正常名稱都測。
- 常見卡點：只測空字串，沒測 null。

### 主線專案銜接
今天補 null 防護。步驟：啟用 nullable、補 validator、補 API integration test。檢查：null 回 400、service 不吃 null、測試可重跑。

### 當天做完後檢查
- [ ] nullable 啟用。
- [ ] null 有測。
- [ ] 錯誤訊息明確。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Request | API DTO | 接 JSON |
| Validator | application | 擋 null |
| Test | unit / integration | 驗證行為 |

### 完整實作流程、範例與注意事項
1. 在 `.csproj` 啟用 `<Nullable>enable</Nullable>`。
2. 為重要輸入補 null 測試。
3. 入口回 400。
```csharp
if (string.IsNullOrWhiteSpace(command.ItemName))
{
    return ValidationResult.Fail("ItemName is required.");
}
```
注意事項：不要用 null 表示「沒有折扣」與「尚未計算」兩種不同狀態。

### 如果結果和預期不同
若 compiler 沒警告，確認專案是否啟用 nullable。

### 負面例子 / 錯誤用法
錯誤做法：`command.ItemName!.Trim()`。問題是壓掉警告但沒有修正風險。修正：先驗證。

### 小練習
替 `Customer.Email` 補 null 測試。

### Junior 常見誤解
誤以為 `!` 是修 bug；它只是告訴 compiler 不要提醒你。

### 一句話總結
Null 要在邊界被說清楚，不要留到深層爆炸。

## Day 16 「聽從你的蜥蜴腦」If

### 這篇文章主要在講什麼
複雜 if 常代表規則需要拆成更清楚的策略或 policy。

### 為什麼需要這個概念
大量 if 會讓測試案例爆炸，新增規則容易影響舊規則。

### 學完這篇你應該會做到什麼
能用策略物件拆折扣規則。

### 核心重點
- 先用測試保護分支。
- 再抽 strategy / policy。
- 不要為了設計模式而設計模式。

### 真實工作流程例子
- 工作任務：不同促銷活動折扣規則開始變多。
- 你先判斷：折扣規則可拆成 `IDiscountRule`。
- 會動到：`IDiscountRule`、多個 rule、policy tests。
- 資料怎麼流：order -> rules -> selected discount。
- 流程路線圖：order -> rule collection -> calculate -> result。
- 工作中會寫 / 檢查的片段：
```csharp
public interface IDiscountRule { bool IsMatch(Order order); decimal Calculate(Order order); }
```
- 交付前驗證：每條 rule 獨立測；組合 policy 測優先順序。
- 常見卡點：規則還很少就過度抽象。

### 主線專案銜接
今天拆折扣 if。步驟：補原行為測試、抽 interface、逐條搬 rule。檢查：舊測試通過、新 rule 容易加、沒有重複 if。

### 當天做完後檢查
- [ ] if 分支有測試。
- [ ] 抽象有實際減少複雜度。
- [ ] 優先順序有測。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Interface | `IDiscountRule` | 規則契約 |
| Rules | `VipDiscountRule` | 個別規則 |
| Policy | `DiscountPolicy` | 組合規則 |

### 完整實作流程、範例與注意事項
1. 先保護原 if 行為。
2. 抽 `IDiscountRule`。
3. 用 collection 套用規則。
```csharp
var rule = _rules.FirstOrDefault(x => x.IsMatch(order));
return rule?.Calculate(order) ?? 0m;
```
注意事項：若只有兩個簡單分支，保留 if 可能更清楚。

### 如果結果和預期不同
若折扣套錯，檢查 rule 順序與 `IsMatch` 條件。

### 負面例子 / 錯誤用法
錯誤做法：把所有促銷都塞在 200 行 if。問題是新增規則會影響舊規則。修正：拆 policy。

### 小練習
新增生日折扣 rule。

### Junior 常見誤解
誤以為看到 if 就要消滅；簡單 if 可以保留。

### 一句話總結
複雜 if 先用測試圈住，再用設計改善。

## Day 17 「提槍上陣」State 設計模式

### 這篇文章主要在講什麼
當狀態轉移規則變複雜時，可以用 State pattern 管理。

### 為什麼需要這個概念
訂單狀態越多，`switch Status` 會越難維護。

### 學完這篇你應該會做到什麼
能用測試保護狀態轉移並評估是否需要 State pattern。

### 核心重點
- 狀態轉移先測。
- State pattern 適合狀態行為差異大時。
- 不要為少量狀態過度設計。

### 真實工作流程例子
- 工作任務：訂單 Pending、Paid、Shipped、Cancelled 行為不同。
- 你先判斷：狀態行為開始分散，可評估 State pattern。
- 會動到：`IOrderState`、各 state class、`Order`。
- 資料怎麼流：order 操作 -> state object -> 更新狀態。
- 流程路線圖：command -> order -> current state -> transition。
- 工作中會寫 / 檢查的片段：
```csharp
public interface IOrderState { void Pay(Order order); void Ship(Order order); }
```
- 交付前驗證：每個狀態的合法與非法操作都測。
- 常見卡點：狀態 class 抽出後 EF mapping 變複雜。

### 主線專案銜接
今天評估並局部導入 state。步驟：先補 switch 行為測試、抽 interface、搬一個狀態。檢查：轉移通過、非法操作失敗、持久化策略清楚。

### 當天做完後檢查
- [ ] 狀態轉移表已列出。
- [ ] 合法/非法都有測。
- [ ] 知道是否真的需要 State pattern。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| State | `IOrderState` | 狀態行為契約 |
| Entity | `Order` | 委派操作 |
| Tests | `OrderStateTests` | 驗證轉移 |

### 完整實作流程、範例與注意事項
1. 列狀態轉移表。
2. 補測試。
3. 抽 state class。
```csharp
public sealed class PendingState : IOrderState
{
    public void Pay(Order order) => order.MarkAsPaid();
    public void Ship(Order order) => throw new InvalidOperationException("Pay first.");
}
```
注意事項：State pattern 會增加類別數，規則不複雜時不一定划算。

### 如果結果和預期不同
若狀態沒有保存，檢查 entity 儲存的是 enum 還是 state class。

### 負面例子 / 錯誤用法
錯誤做法：為兩個狀態建立十幾個 state class。問題是抽象成本大於收益。修正：先保留 enum + method。

### 小練習
替 `Cancelled` 狀態補「不可付款」測試。

### Junior 常見誤解
誤以為設計模式一定比較好；模式要解決真實痛點。

### 一句話總結
State pattern 是給複雜狀態行為的工具，不是 enum 的自動替代品。

## Day 18 「春暖鴨先知」TDD 來了

### 這篇文章主要在講什麼
TDD 的節奏是 Red、Green、Refactor。

### 為什麼需要這個概念
先寫測試能逼你先想使用方式與預期行為。

### 學完這篇你應該會做到什麼
能用 TDD 寫一個小規則。

### 核心重點
- Red：先看到失敗。
- Green：最小實作。
- Refactor：保持行為不變改善設計。

### 真實工作流程例子
- 工作任務：密碼至少 8 碼。
- 你先判斷：這是純規則，先寫 unit test。
- 會動到：`PasswordValidator`、`PasswordValidatorTests`。
- 資料怎麼流：password string -> validator -> bool / result。
- 流程路線圖：test -> fail -> code -> pass -> refactor。
- 工作中會寫 / 檢查的片段：
```csharp
Assert.False(validator.IsValid("abc123"));
```
- 交付前驗證：短密碼失敗；8 碼密碼成功。
- 常見卡點：一次寫太多測試，還沒 green 就迷路。

### 主線專案銜接
今天用 TDD 建 `PasswordValidator`，作為後續會員功能基礎。步驟：寫短密碼測試、最小實作、補 8 碼測試。檢查：先紅後綠、每次只推一小步、重構後仍綠。

### 當天做完後檢查
- [ ] 看過測試失敗。
- [ ] 最小實作通過。
- [ ] 有重構步驟。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Validator | `PasswordValidator` | 密碼規則 |
| Test | `PasswordValidatorTests` | TDD 驅動 |
| CLI | `dotnet test` | 回饋 |

### 完整實作流程、範例與注意事項
1. 寫失敗測試。
2. 寫最小 code。
3. 補下一個案例。
```csharp
public bool IsValid(string password) => password.Length >= 8;
```
注意事項：第一版可以很笨，等測試保護後再整理。

### 如果結果和預期不同
若測試一開始就綠，代表測試可能沒測到新行為。

### 負面例子 / 錯誤用法
錯誤做法：先寫完整 validator 再補測試。問題是測試不會引導設計。修正：先寫一個失敗測試。

### 小練習
新增「必須包含數字」規則。

### Junior 常見誤解
誤以為 TDD 要一次想完整架構；其實它強調小步。

### 一句話總結
TDD 用測試逼你一次只處理一個行為。

## Day 19 「完美不完美」TDD 的困難之處

### 這篇文章主要在講什麼
TDD 難在切小步、取捨設計與忍受初版不完美。

### 為什麼需要這個概念
如果每一步都想做完美，就很難進入 Red-Green-Refactor。

### 學完這篇你應該會做到什麼
能接受先寫簡單實作，再靠測試安全重構。

### 核心重點
- Green 階段可以先簡單。
- Refactor 階段再改善。
- 不要把設計焦慮放進 Red 階段。

### 真實工作流程例子
- 工作任務：優惠券只允許一次使用。
- 你先判斷：先測單一 coupon 狀態，不先建完整促銷平台。
- 會動到：`Coupon`、`CouponTests`。
- 資料怎麼流：coupon.Use -> status change。
- 流程路線圖：first test -> simple state -> second test -> refactor。
- 工作中會寫 / 檢查的片段：
```csharp
coupon.Use();
Assert.True(coupon.IsUsed);
```
- 交付前驗證：第一次可用；第二次丟錯。
- 常見卡點：還沒測完就先設計一堆抽象。

### 主線專案銜接
今天把 coupon 規則用 TDD 加入。步驟：測第一次使用、測重複使用、重構錯誤訊息。檢查：規則集中、狀態清楚、測試小而穩。

### 當天做完後檢查
- [ ] 每次只加一個案例。
- [ ] 沒有先做大型抽象。
- [ ] 重構後測試仍通過。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Entity | `Coupon` | 使用狀態 |
| Test | `CouponTests` | 驅動規則 |
| Service | `ApplyCouponService` | 後續呼叫 |

### 完整實作流程、範例與注意事項
1. 寫第一次使用測試。
2. 寫重複使用測試。
3. 把錯誤語意整理。
```csharp
public void Use()
{
    if (IsUsed) throw new InvalidOperationException("Coupon already used.");
    IsUsed = true;
}
```
注意事項：TDD 不保證設計自動變好，仍需要你在 refactor 做判斷。

### 如果結果和預期不同
若第二次使用沒丟錯，確認 `IsUsed` 是否真的被更新。

### 負面例子 / 錯誤用法
錯誤做法：第一天就建立 CouponFactory、CouponStrategy、CouponEngine。問題是需求還不明確。修正：先完成可驗證的最小規則。

### 小練習
新增「過期 coupon 不可使用」測試。

### Junior 常見誤解
誤以為 TDD 產出的第一版 code 應該很漂亮；漂亮是 refactor 的工作。

### 一句話總結
TDD 允許你先不完美，但不允許你沒有回饋。

## Day 20 「就是真誠」TDD 實彈演習

### 這篇文章主要在講什麼
用實際題目練 TDD，讓測試推動下一步設計。

### 為什麼需要這個概念
只懂概念不練習，很容易回到先寫 code 後補測試。

### 學完這篇你應該會做到什麼
能用 TDD 完成一個小型 domain rule。

### 核心重點
- 從最小案例開始。
- 每個新測試都推動一點行為。
- 重構不改行為。

### 真實工作流程例子
- 工作任務：訂單滿額升級 VIP。
- 你先判斷：這是 customer policy，不是 API 層規則。
- 會動到：`CustomerUpgradePolicy`、tests。
- 資料怎麼流：total amount -> policy -> customer level。
- 流程路線圖：case 1 -> code -> case 2 -> refactor。
- 工作中會寫 / 檢查的片段：
```csharp
Assert.Equal(CustomerLevel.Vip, policy.DecideLevel(12000m));
```
- 交付前驗證：11999 不升級；12000 升級。
- 常見卡點：第一個測試就涵蓋太多規則。

### 主線專案銜接
今天用 TDD 完成會員升級 policy。步驟：寫低於門檻、等於門檻、高於門檻。檢查：邊界清楚、policy 可重用、service 只負責呼叫。

### 當天做完後檢查
- [ ] 至少三個 TDD 小步。
- [ ] 邊界案例有測。
- [ ] policy 沒依賴 DB。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Policy | `CustomerUpgradePolicy` | 判斷等級 |
| Tests | policy tests | 邊界案例 |
| Service | customer service | 呼叫 policy |

### 完整實作流程、範例與注意事項
1. 寫 11999 測試。
2. 寫 12000 測試。
3. 抽常數命名。
```csharp
private const decimal VipThreshold = 12000m;
public CustomerLevel DecideLevel(decimal totalAmount) =>
    totalAmount >= VipThreshold ? CustomerLevel.Vip : CustomerLevel.Normal;
```
注意事項：magic number 要在重構階段命名。

### 如果結果和預期不同
若 12000 沒升級，檢查是否用 `>` 而非 `>=`。

### 負面例子 / 錯誤用法
錯誤做法：只測 15000。問題是門檻行為不明。修正：補 11999、12000。

### 小練習
新增 VIP 降級規則，但先寫測試。

### Junior 常見誤解
誤以為 TDD 只適合 kata；小型 domain rule 很適合練。

### 一句話總結
實戰 TDD 的關鍵是讓每個測試都有推進作用。

## Day 21 「事有輕重緩急」Clean Architecture 入門

### 這篇文章主要在講什麼
把核心商業規則與外部框架分離，讓系統更容易測試與修改。

### 為什麼需要這個概念
如果 domain 依賴 ASP.NET Core 或 EF Core，單元測試會變慢且難隔離。

### 學完這篇你應該會做到什麼
能建立 Domain、Application、Infrastructure、Api 分層。

### 核心重點
- 內層不依賴外層。
- interface 可放在 Application，實作放 Infrastructure。
- API 是入口，不是商業規則集中地。

### 真實工作流程例子
- 工作任務：把下單邏輯從 Controller 搬到 Application。
- 你先判斷：Controller 只處理 HTTP，use case 放 Application service。
- 會動到：`PlaceOrderService`、endpoint、DI。
- 資料怎麼流：HTTP -> command -> service -> repository interface。
- 流程路線圖：request -> API -> Application -> Domain -> Infrastructure。
- 工作中會寫 / 檢查的片段：
```csharp
builder.Services.AddScoped<IOrderRepository, EfOrderRepository>();
```
- 交付前驗證：service unit test 不啟動 web；API integration test 仍通。
- 常見卡點：把 EF Core `DbContext` 直接注入 domain。

### 主線專案銜接
今天建立分層骨架。步驟：拆專案、移動 service、設定 reference。檢查：Domain 無外部套件依賴、Application 可 unit test、Api 負責 DI。

### 當天做完後檢查
- [ ] 專案 reference 方向正確。
- [ ] Domain 不依賴 Infrastructure。
- [ ] Use case 可單元測試。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Domain | domain project | 規則 |
| Application | application project | use case |
| Infrastructure | infrastructure project | EF / external |
| Api | api project | HTTP / DI |

### 完整實作流程、範例與注意事項
1. 建立四個 project。
2. 設定 reference 方向。
3. 搬移 use case。
```text
Api -> Application -> Domain
Infrastructure -> Application -> Domain
```
注意事項：Infrastructure 可以依賴 Application 介面，Application 不依賴 Infrastructure 實作。

### 如果結果和預期不同
若 reference 循環，表示某層責任放錯。

### 負面例子 / 錯誤用法
錯誤做法：Domain entity 使用 `DbContext`。問題是 domain 無法純測。修正：資料存取留在 repository 實作。

### 小練習
畫出目前專案 reference 圖。

### Junior 常見誤解
誤以為 Clean Architecture 是資料夾命名；真正重點是依賴方向。

### 一句話總結
核心規則越少依賴外界，測試越容易。

## Day 22 「戲如人生」Clean Architecture 分層案例

### 這篇文章主要在講什麼
用真實流程看分層責任如何切。

### 為什麼需要這個概念
分層不是多建幾個 project，而是讓每層只處理自己的變動原因。

### 學完這篇你應該會做到什麼
能判斷一段 code 應放 API、Application、Domain 或 Infrastructure。

### 核心重點
- HTTP 細節在 API。
- 流程編排在 Application。
- 商業不變條件在 Domain。
- 外部實作在 Infrastructure。

### 真實工作流程例子
- 工作任務：下單 API 要新增庫存檢查。
- 你先判斷：庫存查詢 interface 在 Application；實作在 Infrastructure；是否可下單由 Application 編排。
- 會動到：`IInventoryService`、`PlaceOrderService`、`InventoryApiClient`。
- 資料怎麼流：command -> inventory interface -> external client -> decision。
- 流程路線圖：API -> Application -> Inventory interface -> Infrastructure client。
- 工作中會寫 / 檢查的片段：
```csharp
public interface IInventoryService { Task<bool> HasStockAsync(string itemName, CancellationToken ct); }
```
- 交付前驗證：有庫存成功；無庫存不建立訂單。
- 常見卡點：直接在 Controller 呼叫外部庫存 API。

### 主線專案銜接
今天加庫存邊界。步驟：定義 interface、service 使用、Infrastructure 實作。檢查：Application test 可 mock 庫存、API 不知道外部細節、無庫存路徑有測。

### 當天做完後檢查
- [ ] 責任放對層。
- [ ] 外部依賴可替換。
- [ ] 無庫存有測試。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Interface | Application | 庫存契約 |
| Client | Infrastructure | 外部 API |
| Service | Application | 編排流程 |

### 完整實作流程、範例與注意事項
1. 在 Application 定義 interface。
2. Service 呼叫 interface。
3. Infrastructure 實作。
```csharp
if (!await _inventory.HasStockAsync(command.ItemName, cancellationToken))
{
    throw new InvalidOperationException("Out of stock.");
}
```
注意事項：不要讓外部 API DTO 流進 domain。

### 如果結果和預期不同
若 unit test 需要真庫存 API，代表抽象邊界沒切好。

### 負面例子 / 錯誤用法
錯誤做法：Controller 裡直接 `new HttpClient()` 查庫存。問題是難測且耦合。修正：抽 `IInventoryService`。

### 小練習
替庫存不足補 API integration test。

### Junior 常見誤解
誤以為 interface 要放實作旁邊；在 Clean Architecture 中常放需要它的內層。

### 一句話總結
分層的目的，是讓變動停在該停的地方。

## Day 23 「啟動！Outside-In 之路」Controller 與單元測試

### 這篇文章主要在講什麼
Controller 或 endpoint 應保持薄，測試重點是 HTTP 邊界。

### 為什麼需要這個概念
Controller 太胖會把 HTTP、驗證、流程、商業規則混在一起。

### 學完這篇你應該會做到什麼
能用 integration test 驗證 endpoint 行為。

### 核心重點
- Controller 不放商業規則。
- 薄 Controller 可用 integration test 保護。
- request / response shape 很重要。

### 真實工作流程例子
- 工作任務：`POST /orders` 要回 `201 Created`。
- 你先判斷：HTTP status 是 API 層行為，用 integration test。
- 會動到：endpoint、request DTO、integration test。
- 資料怎麼流：JSON -> DTO -> service -> response。
- 流程路線圖：HTTP request -> endpoint -> service -> response。
- 工作中會寫 / 檢查的片段：
```csharp
var response = await client.PostAsJsonAsync("/orders", request);
Assert.Equal(HttpStatusCode.Created, response.StatusCode);
```
- 交付前驗證：成功 201；validation error 400。
- 常見卡點：只測 service，忘記 route 寫錯也會讓 API 壞。

### 主線專案銜接
今天補下單 endpoint test。步驟：建立 `WebApplicationFactory`、送 request、assert status。檢查：route 正確、JSON mapping 正確、錯誤碼正確。

### 當天做完後檢查
- [ ] endpoint 有 integration test。
- [ ] Controller / endpoint 很薄。
- [ ] HTTP response 有測。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Endpoint | `Program.cs` | HTTP 入口 |
| DTO | `PlaceOrderRequest` | request |
| Test | `OrderEndpointTests` | HTTP 驗證 |

### 完整實作流程、範例與注意事項
1. 加 `Microsoft.AspNetCore.Mvc.Testing`。
2. 建 `WebApplicationFactory<Program>`。
3. 呼叫 endpoint。
```csharp
public partial class Program { }
```
注意事項：Minimal API 的 `Program` 若是 internal，測試可能需要 partial public class。

### 如果結果和預期不同
若 404，先檢查 route 與 HTTP method。

### 負面例子 / 錯誤用法
錯誤做法：只測 Controller method，沒有測 route。問題是路由錯誤不會發現。修正：加 integration test。

### 小練習
測 `GET /orders/{id}` 找不到時回 404。

### Junior 常見誤解
誤以為 Controller test 一定要 mock HTTP；ASP.NET Core 可以用 TestServer。

### 一句話總結
API 邊界要用接近真實 HTTP 的方式驗證。

## Day 24 「小步快跑」Service 與單元測試（上）

### 這篇文章主要在講什麼
Application Service 負責 use case 流程編排，是單元測試重點。

### 為什麼需要這個概念
Service 是需求變動最常碰到的地方，沒有測試會很難改。

### 學完這篇你應該會做到什麼
能測 service 成功流程。

### 核心重點
- Service 測流程。
- 外部依賴 mock。
- Domain 規則不要塞滿 service。

### 真實工作流程例子
- 工作任務：下單成功要付款、存訂單、發事件。
- 你先判斷：這是 service orchestration。
- 會動到：`PlaceOrderService`、mocks、tests。
- 資料怎麼流：command -> payment -> order -> repository -> event。
- 流程路線圖：command -> service -> gateway/repository/publisher。
- 工作中會寫 / 檢查的片段：
```csharp
repository.Verify(x => x.SaveAsync(It.IsAny<Order>(), It.IsAny<CancellationToken>()), Times.Once);
```
- 交付前驗證：三個協作者都有正確互動。
- 常見卡點：service 測試太關心內部 helper。

### 主線專案銜接
今天完成下單成功流程測試。步驟：mock payment、mock repository、mock publisher。檢查：付款一次、存一次、發事件一次。

### 當天做完後檢查
- [ ] 成功流程有測。
- [ ] mock 只放外部依賴。
- [ ] service 不含 HTTP 細節。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Service | Application | use case |
| Gateway | Interface mock | 外部付款 |
| Repository | Interface mock | 儲存 |

### 完整實作流程、範例與注意事項
1. 準備 command。
2. setup mock。
3. 呼叫 service。
4. verify 重要副作用。
```csharp
await service.PlaceOrderAsync(new("book", 500m), CancellationToken.None);
```
注意事項：不要 verify 每個微小 private 呼叫。

### 如果結果和預期不同
若 verify 沒通，檢查 mock 是否傳進 service constructor。

### 負面例子 / 錯誤用法
錯誤做法：測 service 時啟動整個 WebApplicationFactory。問題是變成 integration test。修正：service unit test 直接 new service。

### 小練習
新增「成功下單後發送通知」測試。

### Junior 常見誤解
誤以為 service test 要測資料庫；資料庫交給 integration test。

### 一句話總結
Service unit test 要聚焦 use case 的流程與決策。

## Day 25 「行禮如儀？行將就木？」Service 與單元測試（下）

### 這篇文章主要在講什麼
Service 測試要避免只測實作細節，應保護可觀察行為。

### 為什麼需要這個概念
測太細會讓重構成本高，測太粗又保護不到流程。

### 學完這篇你應該會做到什麼
能挑選 service 測試的驗證點。

### 核心重點
- 驗證重要輸出與副作用。
- 少測內部步驟。
- 測錯誤路徑。

### 真實工作流程例子
- 工作任務：付款失敗時不應發通知。
- 你先判斷：通知是付款成功後副作用。
- 會動到：`INotificationSender`、service tests。
- 資料怎麼流：payment false -> stop -> no notification。
- 流程路線圖：command -> payment fail -> exception -> no save/no notify。
- 工作中會寫 / 檢查的片段：
```csharp
notification.Verify(x => x.SendAsync(It.IsAny<string>(), It.IsAny<CancellationToken>()), Times.Never);
```
- 交付前驗證：失敗不通知；成功通知一次。
- 常見卡點：只 assert exception，忘記驗證副作用沒發生。

### 主線專案銜接
今天補失敗流程副作用測試。步驟：setup payment false、assert throw、verify never。檢查：不存、不發、不回成功。

### 當天做完後檢查
- [ ] 失敗流程有測。
- [ ] `Times.Never` 用在關鍵副作用。
- [ ] 測試不綁 private helper。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Service | Application | 決策 |
| Notification | mock | 副作用 |
| Test | service tests | 驗證不發生 |

### 完整實作流程、範例與注意事項
1. 設定付款失敗。
2. 呼叫 service。
3. 驗證不通知。
```csharp
await Assert.ThrowsAsync<InvalidOperationException>(() => service.PlaceOrderAsync(command, ct));
```
注意事項：`Times.Never` 適合重要副作用，不要濫用在所有方法。

### 如果結果和預期不同
若通知仍發出，檢查 service 是否在付款前就發通知。

### 負面例子 / 錯誤用法
錯誤做法：verify service 內部每個 helper 的呼叫順序。問題是重構即壞。修正：驗證外部可觀察行為。

### 小練習
測庫存不足時不付款。

### Junior 常見誤解
誤以為 verify 越多越完整；verify 應該對準重要合約。

### 一句話總結
好的 service 測試保護行為，不綁死寫法。

## Day 26 「一個巨星的誕生」Entity、Repository 與單元測試

### 這篇文章主要在講什麼
Entity 適合 unit test；Repository 更適合 integration test。

### 為什麼需要這個概念
Repository 真正風險在 SQL、mapping、transaction，用 mock 很難測到。

### 學完這篇你應該會做到什麼
能為 Entity 與 Repository 選擇不同測試策略。

### 核心重點
- Entity 測商業規則。
- Repository 測真 provider 行為。
- 不建議 mock `DbSet` 當主要驗證。

### 真實工作流程例子
- 工作任務：確認訂單能寫入資料庫並查回。
- 你先判斷：這是 Repository integration test。
- 會動到：`EfOrderRepository`、DbContext、integration test。
- 資料怎麼流：repository.Save -> DbContext -> DB -> query。
- 流程路線圖：test -> repository -> db -> assert row。
- 工作中會寫 / 檢查的片段：
```csharp
await repository.SaveAsync(order, CancellationToken.None);
Assert.NotNull(await repository.FindAsync(order.Id, CancellationToken.None));
```
- 交付前驗證：可寫入、可查回、mapping 正確。
- 常見卡點：用 InMemory provider 測到通，但真 SQL provider 壞。

### 主線專案銜接
今天新增 repository integration test。步驟：建立測試 DB、寫入 order、查回。檢查：欄位正確、交易正常、測試資料隔離。

### 當天做完後檢查
- [ ] Entity unit test 與 Repository integration test 分開。
- [ ] DB 測試可重跑。
- [ ] 不 mock `DbSet` 當主要測試。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Entity | Domain | 規則 |
| Repository | Infrastructure | 持久化 |
| Test | IntegrationTests | 驗證 DB |

### 完整實作流程、範例與注意事項
1. 準備測試 database。
2. 建 repository。
3. Save 後 Find。
```csharp
var saved = await repository.FindAsync(order.Id, CancellationToken.None);
Assert.Equal(order.Amount, saved!.Amount);
```
注意事項：測試資料要隔離，避免測試互相污染。

### 如果結果和預期不同
若查不到資料，檢查是否呼叫 `SaveChangesAsync`。

### 負面例子 / 錯誤用法
錯誤做法：mock `DbSet` 後宣稱 Repository 沒問題。問題是 mapping 與 SQL 沒測到。修正：使用 integration test。

### 小練習
替 `FindByCustomerId` 補 repository integration test。

### Junior 常見誤解
誤以為所有東西都該 unit test；有些風險需要 integration test。

### 一句話總結
Entity 測規則，Repository 測它真的能和資料庫合作。

## Day 27 「能省則省」Clean Architecture ft. CQRS

### 這篇文章主要在講什麼
Command 與 Query 可以分開設計，讓寫入流程與讀取模型各自清楚。

### 為什麼需要這個概念
讀取常追求資料 shape，寫入追求規則與一致性，硬塞一起會複雜。

### 學完這篇你應該會做到什麼
能分別測 command handler 與 query handler。

### 核心重點
- Command 改變狀態。
- Query 不改變狀態。
- 測試策略可不同。

### 真實工作流程例子
- 工作任務：訂單列表需要顯示總金額與狀態文字。
- 你先判斷：這是 query model，不必載入完整 domain 行為。
- 會動到：`GetOrdersQueryHandler`、query test、DB。
- 資料怎麼流：query -> repository/read db -> DTO list。
- 流程路線圖：HTTP GET -> query handler -> read model -> response。
- 工作中會寫 / 檢查的片段：
```csharp
public sealed record OrderListItem(Guid Id, decimal Amount, string StatusText);
```
- 交付前驗證：查詢不改 DB；資料 shape 正確。
- 常見卡點：為了列表載入完整 aggregate，效能差又難測。

### 主線專案銜接
今天新增訂單列表 query。步驟：定義 DTO、寫 query handler、補 integration test。檢查：欄位正確、排序正確、無副作用。

### 當天做完後檢查
- [ ] Command / Query 分清楚。
- [ ] Query 不改狀態。
- [ ] DTO shape 有測。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Command | Application | 寫入流程 |
| Query | Application | 讀取流程 |
| DTO | Contracts | 回傳 shape |

### 完整實作流程、範例與注意事項
1. 建 query DTO。
2. 寫 query handler。
3. 用 integration test 驗證資料 shape。
```csharp
var items = await handler.HandleAsync(new GetOrdersQuery(), CancellationToken.None);
Assert.All(items, x => Assert.False(string.IsNullOrWhiteSpace(x.StatusText)));
```
注意事項：CQRS 不等於一定要上 message bus 或 event sourcing。

### 如果結果和預期不同
若 query 造成資料異動，檢查 handler 是否誤用 command service。

### 負面例子 / 錯誤用法
錯誤做法：簡單 CRUD 也拆十幾層 CQRS。問題是成本過高。修正：在複雜度需要時再拆。

### 小練習
替 `GetOrderDetailQuery` 補測試。

### Junior 常見誤解
誤以為 CQRS 是微服務專用；在單體內也可用輕量分離。

### 一句話總結
讀寫分離的價值，是讓不同目的的 code 不互相拖累。

## Day 28 「最好避免犯錯的方法」單元測試與 GitFlow、主線開發

### 這篇文章主要在講什麼
測試讓團隊能更頻繁、安全地合併程式。

### 為什麼需要這個概念
長期分支容易累積衝突與未知風險。

### 學完這篇你應該會做到什麼
能在 PR 與 CI 中安排測試。

### 核心重點
- 小步提交。
- PR 自動跑測試。
- 測試失敗不合併。

### 真實工作流程例子
- 工作任務：每個 PR 必須跑 `dotnet test`。
- 你先判斷：這是 CI pipeline 責任，不是靠 reviewer 手動提醒。
- 會動到：`.github/workflows/dotnet-test.yml`。
- 資料怎麼流：push -> GitHub Actions -> restore/build/test -> status。
- 流程路線圖：PR -> CI -> test report -> merge gate。
- 工作中會寫 / 檢查的片段：
```yaml
- name: Test
  run: dotnet test --configuration Release
```
- 交付前驗證：PR 會觸發；測試失敗會擋合併。
- 常見卡點：本機有跑，CI 因環境不同失敗。

### 主線專案銜接
今天加入 CI。步驟：建立 workflow、跑 restore/build/test、設定 branch protection。檢查：PR 有 status、失敗不能 merge、log 可讀。

### 當天做完後檢查
- [ ] CI 會跑測試。
- [ ] 失敗擋合併。
- [ ] 測試 log 可追。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| Workflow | `.github/workflows` | 自動測試 |
| Test | `dotnet test` | 驗證 |
| GitHub | branch protection | 合併門檻 |

### 完整實作流程、範例與注意事項
1. 建 workflow。
2. 設定 .NET SDK。
3. 執行 test。
```yaml
name: dotnet-test
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'
      - run: dotnet test --configuration Release
```
注意事項：integration test 若需要 DB，要在 CI 補 service container 或測試資料庫。

### 如果結果和預期不同
若 CI 找不到 SDK，檢查 `dotnet-version` 與專案 target framework。

### 負面例子 / 錯誤用法
錯誤做法：測試失敗仍允許 merge。問題是 main branch 失去信任。修正：設定 required checks。

### 小練習
讓 workflow 分開跑 unit 與 integration。

### Junior 常見誤解
誤以為 CI 是 DevOps 的事；開發者要能讀懂測試失敗 log。

### 一句話總結
自動測試是團隊合併程式的安全門。

## Day 29 「Try it!」單元測試與軟體工程

### 這篇文章主要在講什麼
測試是整體工程習慣的一部分，包含設計、review、CI、交付。

### 為什麼需要這個概念
只會寫測試 API，不代表團隊真的能靠測試降低風險。

### 學完這篇你應該會做到什麼
能在 code review 中檢查測試品質。

### 核心重點
- 測試要對準風險。
- Review 要看測試是否保護行為。
- 測試命名與資料也需要品質。

### 真實工作流程例子
- 工作任務：review 同事的折扣 PR。
- 你先判斷：先看需求風險與測試是否覆蓋邊界。
- 會動到：PR diff、測試檔、CI log。
- 資料怎麼流：需求 -> code change -> tests -> CI result。
- 流程路線圖：PR -> review tests -> run CI -> feedback。
- 工作中會寫 / 檢查的片段：
```text
Review comment: 請補 amount = 1000 的邊界測試，避免免運門檻被誤改。
```
- 交付前驗證：新規則、舊規則、邊界都被測。
- 常見卡點：只看 production code，沒看測試是否真的會失敗。

### 主線專案銜接
今天建立 review checklist。步驟：列風險、看測試、跑 CI。檢查：測試名稱清楚、錯誤路徑有測、沒有只追 coverage。

### 當天做完後檢查
- [ ] Review 有看測試。
- [ ] 測試能對應需求。
- [ ] 邊界案例有被問到。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| PR | GitHub | 變更 |
| Tests | test files | 風險保護 |
| CI | Actions | 自動驗證 |

### 完整實作流程、範例與注意事項
1. 讀需求。
2. 看 production diff。
3. 看測試是否能讓錯誤失敗。
4. 留具體 review comment。
```text
Checklist: happy path / edge case / error path / regression / readable names
```
注意事項：不要只要求「補測試」，要說明補哪個風險。

### 如果結果和預期不同
若測試全綠但你覺得不安心，嘗試在本機故意改錯 production code 看測試是否會紅。

### 負面例子 / 錯誤用法
錯誤做法：只看 coverage 從 70% 到 80%。問題是沒有確認保護力。修正：看測試是否對準行為。

### 小練習
找一個舊 PR，補一則更具體的測試 review comment。

### Junior 常見誤解
誤以為測試是寫完就好；測試也需要 review。

### 一句話總結
測試品質是工程品質的一部分。

## Day 30 「無心之心，道之所存」結語

### 這篇文章主要在講什麼
測試的終點不是工具熟練，而是讓系統保持可理解、可修改、可交付。

### 為什麼需要這個概念
如果測試變成形式，團隊仍然不敢改 code。

### 學完這篇你應該會做到什麼
能為自己的專案建立測試改善路線。

### 核心重點
- 先保護高風險行為。
- 讓測試成為日常流程。
- 持續整理測試。

### 真實工作流程例子
- 工作任務：既有專案幾乎沒有測試，要開始補。
- 你先判斷：不要一次補全；先找最常改、最常壞、最有商業價值的流程。
- 會動到：測試專案、CI、核心 service tests。
- 資料怎麼流：需求風險 -> 測試清單 -> 補測 -> CI。
- 流程路線圖：盤點風險 -> 選 use case -> 補測 -> 重構 -> CI。
- 工作中會寫 / 檢查的片段：
```text
Top 3 first tests: PlaceOrder success, Payment failed, Repository save/find.
```
- 交付前驗證：新測試能在 CI 跑；團隊知道如何新增測試。
- 常見卡點：想一次導入完整測試文化，最後沒有落地。

### 主線專案銜接
今天做總驗收。步驟：跑全部測試、檢查 CI、列下一輪改善。檢查：unit/integration 都可跑、README 有測試指令、重要 use case 有保護。

### 當天做完後檢查
- [ ] 有測試改善清單。
- [ ] 有 CI。
- [ ] 團隊知道測試規則。

### 範例範圍地圖
| 部分 | 位置 | 責任 |
| --- | --- | --- |
| README | repo root | 測試指令 |
| CI | workflow | 自動驗證 |
| Tests | tests folder | 行為保護 |

### 完整實作流程、範例與注意事項
1. 跑 `dotnet test`。
2. 確認 CI 綠燈。
3. 在 README 記錄測試策略。
```powershell
dotnet test --configuration Release
```
注意事項：測試策略要隨專案調整，不是一次設定後永不更新。

### 如果結果和預期不同
若測試太慢，先分類 unit / integration，再找最慢測試原因。

### 負面例子 / 錯誤用法
錯誤做法：只訂 coverage KPI。問題是大家可能寫無意義測試。修正：用重要行為與風險清單驅動。

### 小練習
為自己的專案列出前三個最該補測試的 use case。

### Junior 常見誤解
誤以為測試學完就是會某個框架；真正能力是知道什麼風險該怎麼驗證。

### 一句話總結
測試的價值，是讓團隊持續有勇氣改進系統。

## 實務使用情境

- 修改高風險商業規則前，先補 characterization test。
- 新增 application service 時，同步補成功與失敗路徑。
- API route、status code、JSON shape 用 integration test 保護。
- Repository 與 EF Core mapping 用 integration test，不只 mock。
- PR 與 CI 以測試作為合併門檻。

## 不適合使用的情境

- 還在探索 throw-away prototype，測試可少量聚焦核心假設。
- UI 純視覺排版細節不一定用 unit test，可能用 snapshot、visual regression 或人工驗收。
- 第三方套件本身行為不用重測，應測你的整合方式。
- 很簡單且短期丟棄的 script 不必套完整架構。

## 參考資料

- iThome 鐵人賽系列頁：<https://ithelp.ithome.com.tw/2020-12th-ironman/articles/3897>
- .NET 官方支援原則：<https://dotnet.microsoft.com/en-us/platform/support/policy>
- .NET 10 發布公告：<https://devblogs.microsoft.com/dotnet/announcing-dotnet-10/>
- xUnit.net Core Framework v3 3.2.2 release notes：<https://xunit.net/releases/v3/3.2.2>
- ASP.NET Core integration tests：<https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-10.0>
