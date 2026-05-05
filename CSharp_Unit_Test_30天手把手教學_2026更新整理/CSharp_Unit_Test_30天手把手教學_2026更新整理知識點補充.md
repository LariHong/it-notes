# C# Unit Test 30 天手把手教學 2026 更新整理 知識點補充

## Day 1：「科學 v.s. 數學」前言

### Visual Studio 內如何使用

#### 1. 知識點

在 Visual Studio 裡寫 C# 單元測試，核心流程是：同一個 Solution 裡放「正式專案」與「測試專案」，測試專案參考正式專案，然後用 Test Explorer 找到並執行 `[Fact]` 測試。

你可以把 Visual Studio 想成三個視窗一起配合：

- Solution Explorer：管理正式專案、測試專案、檔案與專案參考。
- Code Editor：寫正式程式與測試程式。
- Test Explorer：執行測試、看紅燈綠燈、點失敗訊息回到問題行。

Day 1 的重點不是背工具，而是建立「改 code 前後都能按一次測試」的工作節奏。

#### 2. 使用注意

- 如果 Test Explorer 沒看到測試，先 Build Solution，Visual Studio 通常要編譯後才會探索測試。
- 如果還是看不到，檢查測試方法是否有 `[Fact]`，測試類別與方法是否是 `public`。
- 測試專案要安裝 xUnit 與 Visual Studio runner，例如 `xunit`、`xunit.runner.visualstudio`、`Microsoft.NET.Test.Sdk`。
- 測試專案要加入正式專案參考，否則測試看不到 `DiscountCalculator`。
- Test Explorer 裡可以 Run All，也可以只選單一測試右鍵 Run 或 Debug。

#### 3. 工作上常用知識

- 開發時常用 Test Explorer 只跑目前正在改的測試，確認小修改沒有破壞行為。
- Debug 測試時，可以在測試或正式程式碼中下 breakpoint，從測試案例一路追進 production code。
- Code review 時，測試檔常會跟正式程式一起看：正式程式回答「怎麼做」，測試回答「需求期望是什麼」。
- 本機用 Visual Studio 跑測試，CI 通常用 `dotnet test` 跑同一批測試；兩邊應該得到一致結果。

#### 4. 常見誤解

- 誤解：Visual Studio 只能用 MSTest。
  實際上 Test Explorer 可以跑 xUnit、NUnit、MSTest，只要測試專案有對應 adapter。
- 誤解：按 Run All 綠燈就代表程式完全沒問題。
  綠燈只代表目前測試涵蓋的情境通過，沒寫到的情境仍可能壞。
- 誤解：測試一定要從 UI 操作開始。
  單元測試通常直接測小單位邏輯，例如 `DiscountCalculator.Calculate()`，速度才會快。

#### 5. 小範例

假設你已經在 Visual Studio 建好一個 Solution，可以照這個順序做：

1. 在 Solution Explorer 對 Solution 按右鍵，選 Add > New Project。
2. 搜尋 `xUnit Test Project`，建立 `OrderTestingLab.UnitTests`。
3. 對測試專案按右鍵，選 Add > Project Reference。
4. 勾選正式專案，例如 `OrderTestingLab.Domain`。
5. 新增 `DiscountCalculatorTests.cs`。
6. 打開 Test Explorer：Test > Test Explorer。
7. Build Solution。
8. 在 Test Explorer 按 Run All。

```csharp
using Xunit;

public sealed class DiscountCalculatorTests
{
    [Fact]
    public void Calculate_ReturnsVipDiscount()
    {
        var calculator = new DiscountCalculator();

        var discount = calculator.Calculate(CustomerLevel.Vip, 1000m);

        Assert.Equal(100m, discount);
    }
}
```

如果測試失敗，先點 Test Explorer 裡的失敗測試，看錯誤訊息。例如 expected 是 `100`，actual 是 `0`，代表測試有執行，但正式邏輯還沒有符合預期。

### Day 1 手把手範例

#### 1. 知識點

這個範例從零做出一個最小可跑的 C# 單元測試：正式程式只有一個折扣計算器，測試只驗證 VIP 客戶 1000 元訂單應折扣 100 元。

重點是先體驗完整閉環：

```text
寫測試 -> 看到紅燈 -> 補正式程式 -> 看到綠燈
```

這就是 Day 1 說的「用可重複驗證降低不確定性」。

#### 2. 使用注意

- 第一個範例要小，不要一開始就建立 API、資料庫、登入、權限。
- 測試名稱要描述情境，例如 `Calculate_ReturnsVipDiscount`，不要只叫 `Test1`。
- 先讓測試失敗一次，確認測試真的有保護到需求。
- 實作只補到測試通過即可，Day 1 不需要設計完整折扣系統。

#### 3. 工作上常用知識

- 新需求進來時，可以先把需求改寫成測試名稱。
- 修 bug 時，可以先寫一個會重現 bug 的失敗測試，再修正式程式。
- 重構前先跑測試，重構後再跑測試；兩次都綠燈，才比較有信心。
- 當測試失敗時，先看 expected / actual，再看 stack trace，不要急著大改。

#### 4. 常見誤解

- 誤解：一定要先把正式功能全部寫完才能寫測試。
  Day 1 建議反過來，先用一個測試描述你期待的行為。
- 誤解：測試程式只是範例程式。
  測試是專案的一部分，未來每次修改都會再執行。
- 誤解：測試失敗代表你做錯。
  一開始的紅燈是正常流程，它證明測試能抓到尚未完成的行為。

#### 5. 小範例

步驟 1：在正式專案新增 enum。

```csharp
public enum CustomerLevel
{
    Normal,
    Vip
}
```

步驟 2：在測試專案先寫測試。

```csharp
using Xunit;

public sealed class DiscountCalculatorTests
{
    [Fact]
    public void Calculate_ReturnsVipDiscount()
    {
        var calculator = new DiscountCalculator();

        var discount = calculator.Calculate(CustomerLevel.Vip, 1000m);

        Assert.Equal(100m, discount);
    }
}
```

這時候 Build 可能會失敗，因為 `DiscountCalculator` 還不存在。這不是壞事，代表測試已經描述出你缺少的正式程式。

步驟 3：在正式專案新增最小實作。

```csharp
public sealed class DiscountCalculator
{
    public decimal Calculate(CustomerLevel level, decimal amount)
    {
        if (level == CustomerLevel.Vip)
        {
            return amount * 0.1m;
        }

        return 0m;
    }
}
```

步驟 4：回 Visual Studio 執行測試。

```text
Test > Test Explorer > Run All
```

看到綠燈後，再補第二個測試保護一般會員。

```csharp
[Fact]
public void Calculate_ReturnsZeroDiscountForNormalCustomer()
{
    var calculator = new DiscountCalculator();

    var discount = calculator.Calculate(CustomerLevel.Normal, 1000m);

    Assert.Equal(0m, discount);
}
```

這樣 Day 1 的最小練習就完成了：你已經用 Visual Studio 建立測試專案、寫第一個測試、看過測試失敗、補實作、再看測試通過。

## 參考來源

- Microsoft Learn：[Run unit tests by using Test Explorer](https://learn.microsoft.com/en-us/visualstudio/test/run-unit-tests-with-test-explorer?view=visualstudio)
- Microsoft Learn：[Unit test basics](https://learn.microsoft.com/en-us/visualstudio/test/unit-test-basics?view=visualstudio)
- Microsoft Learn：[Create unit test method stubs from code](https://learn.microsoft.com/en-us/visualstudio/test/create-unit-tests-menu?view=visualstudio)
