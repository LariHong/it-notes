# Design Pattern 系列 Day11-Day29 整理

## 這份筆記怎麼讀

這 19 篇文章從建立型模式進入結構型模式，再到行為型模式。讀的時候不要先背 UML，而是先問三件事：這個模式把哪一段變動隔離起來？它讓哪個物件不用知道太多細節？它付出的代價是不是值得？

本筆記依照 article-link-explainer 的規格整理：每篇都保留來源、核心問題、角色對照、C# 範例、錯誤用法與 junior developer 學習建議。C# 範例是為了幫助理解而改寫，概念來自原文，程式碼不是原文逐字轉換。

## 來源清單

| Day | 主題 | 原文 |
| --- | --- | --- |
| Day 11 | Factory Pattern | https://ithelp.ithome.com.tw/articles/10238697 |
| Day 12 | Abstract Factory Pattern | https://ithelp.ithome.com.tw/articles/10239444 |
| Day 13 | Builder Pattern | https://ithelp.ithome.com.tw/articles/10240534 |
| Day 14 | Prototype Pattern | https://ithelp.ithome.com.tw/articles/10240991 |
| Day 15 | Adapter Pattern | https://ithelp.ithome.com.tw/articles/10241632 |
| Day 16 | Bridge Pattern | https://ithelp.ithome.com.tw/articles/10242349 |
| Day 17 | Composite Pattern | https://ithelp.ithome.com.tw/articles/10242929 |
| Day 18 | Decorator Pattern | https://ithelp.ithome.com.tw/articles/10243619 |
| Day 19 | Facade Pattern | https://ithelp.ithome.com.tw/articles/10244254 |
| Day 20 | Flyweight Pattern | https://ithelp.ithome.com.tw/articles/10245053 |
| Day 21 | Proxy Pattern | https://ithelp.ithome.com.tw/articles/10245445 |
| Day 22 | Chain of Responsibility Pattern | https://ithelp.ithome.com.tw/articles/10245941 |
| Day 23 | Command Pattern | https://ithelp.ithome.com.tw/articles/10246587 |
| Day 24 | Iterator Pattern | https://ithelp.ithome.com.tw/articles/10247111 |
| Day 25 | Mediator Pattern | https://ithelp.ithome.com.tw/articles/10247722 |
| Day 26 | State Pattern | https://ithelp.ithome.com.tw/articles/10248228 |
| Day 27 | Memento Pattern | https://ithelp.ithome.com.tw/articles/10248825 |
| Day 28 | Observer Pattern | https://ithelp.ithome.com.tw/articles/10249203 |
| Day 29 | Strategy Pattern | https://ithelp.ithome.com.tw/articles/10249638 |

## 整體地圖

| 類型 | Day | 主要在隔離什麼變動 |
| --- | --- | --- |
| 建立型 | 11-14 | 物件如何被建立、建立流程如何被封裝、既有物件如何被複製 |
| 結構型 | 15-21 | 類別與物件如何組合、轉接、包裝、簡化與共享 |
| 行為型 | 22-29 | 物件之間如何協作、請求如何傳遞、狀態與演算法如何切換 |

---

## Day 11：工廠模式 | Factory Pattern

### 這篇文章主要在講什麼

原文用餐廳點餐說明 Simple Factory 與 Factory Method。Simple Factory 把 `new Steak()`、`new Chicken()` 這類建立物件的邏輯集中到工廠；Factory Method 則再進一步把「建立哪個產品」交給不同的子工廠。

### 為什麼需要這個概念

如果 client 到處直接 `new` 具體類別，新增產品時就會到處修改。工廠模式把建立物件的責任集中或分散到工廠類別，讓使用端只依賴抽象產品。

### 核心重點

- Simple Factory 是常見開發習慣，不一定被視為 GoF pattern。
- Factory Method 是正式設計模式，讓子類別決定要建立哪個具體產品。
- 工廠適合產品有共同抽象，但建立邏輯會變動的情境。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Factory（工廠） | MealFactory | 建立餐點物件 |
| ConcreteFactory（實體工廠） | SteakFactory, ChickenFactory, PorkFactory | 各自建立特定餐點 |
| Product（抽象產品） | CookMeal | 定義餐點共同行為 |
| ConcreteProduct（實體產品） | Steak, Chicken, Pork | 實作實際餐點 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public interface IMeal // Product（抽象產品）：原文 CookMeal
{
    void Cook();
}

public sealed class Steak : IMeal // ConcreteProduct（實體產品）：Steak
{
    public void Cook() => Console.WriteLine("煎牛排");
}

public interface IMealFactory // Factory（抽象工廠）：原文 MealFactory
{
    IMeal Create();
}

public sealed class SteakFactory : IMealFactory // ConcreteFactory（實體工廠）
{
    public IMeal Create() => new Steak();
}

public sealed class MealOrderService
{
    private readonly IMealFactory _factory;

    public MealOrderService(IMealFactory factory) => _factory = factory;

    public void Order()
    {
        IMeal meal = _factory.Create();
        meal.Cook();
    }
}

new MealOrderService(new SteakFactory()).Order();
```

### 程式碼對應概念說明

`MealOrderService` 不知道 `Steak` 怎麼建立，只知道工廠會給它一個 `IMeal`。如果未來新增雞肉，只要新增 `Chicken` 與 `ChickenFactory`。

### 負面例子 / 錯誤用法

錯誤做法是把所有產品都塞進一個巨大 `switch`，最後 Simple Factory 變成需求集中爆炸點。修正方向是：產品種類少時可用 Simple Factory；種類多、建立規則差異大時改用 Factory Method 或 DI container。

### Junior 常見誤解

不要以為「有 new 就一定要工廠」。如果物件很單純、不需要抽象替換，直接建立會更清楚。

### 一句話總結

工廠模式把「建立哪個物件」從使用端移走，降低 client 對具體類別的依賴。

---

## Day 12：抽象工廠模式 | Abstract Factory Pattern

### 這篇文章主要在講什麼

原文延續餐廳例子，把「台式餐廳」與「義式餐廳」視為不同產品族。抽象工廠不是只建立一個產品，而是建立一組彼此相容的產品。

### 為什麼需要這個概念

當系統需要整組切換風格、平台或供應商時，只用 Factory Method 會產生太多零散工廠。抽象工廠把同一族產品集中在同一個工廠中，避免台式牛排配到義式醬料這種不一致。

### 核心重點

- 抽象工廠建立的是產品族，不是單一物件。
- 選定 ConcreteFactory 後，就等於選定整組產品風格。
- 適合 UI theme、跨資料庫 provider、跨雲端供應商 SDK 包裝。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| AbstractFactory | Restaurant | 定義建立肉品與醬料的方法 |
| ConcreteFactory | TWRestaurant, ITRestaurant | 建立台式或義式產品族 |
| Product | CookMeal, Ingredient | 定義餐點與原料抽象 |
| ConcreteProduct | Steak, Pork, TWIngredient, ITIngredient | 實際產品 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public interface IButton { string Render(); } // Product：按鈕
public interface IDialog { string Open(); } // Product：對話框

public sealed class DarkButton : IButton { public string Render() => "Dark button"; }
public sealed class DarkDialog : IDialog { public string Open() => "Dark dialog"; }

public interface IUiFactory // AbstractFactory：建立同一產品族
{
    IButton CreateButton();
    IDialog CreateDialog();
}

public sealed class DarkUiFactory : IUiFactory // ConcreteFactory：深色產品族
{
    public IButton CreateButton() => new DarkButton();
    public IDialog CreateDialog() => new DarkDialog();
}

public sealed class CheckoutPage
{
    private readonly IUiFactory _factory;
    public CheckoutPage(IUiFactory factory) => _factory = factory;

    public void Render()
    {
        Console.WriteLine(_factory.CreateButton().Render());
        Console.WriteLine(_factory.CreateDialog().Open());
    }
}
```

### 程式碼對應概念說明

`CheckoutPage` 不決定要深色或淺色 UI，只依賴 `IUiFactory`。一旦注入 `DarkUiFactory`，整組 UI 元件都會保持同一風格。

### 負面例子 / 錯誤用法

如果系統只有一個產品、沒有產品族一致性問題，使用抽象工廠會過度設計。修正方向是先用 Factory Method 或直接 DI，等真的有整組切換需求再提升抽象層級。

### Junior 常見誤解

Factory Method 像「指定某家工廠做一個產品」；Abstract Factory 像「指定一套供應體系，拿到整組相容產品」。

### 一句話總結

抽象工廠用來建立一整組相容物件，重點是產品族的一致性。

---

## Day 13：生成器模式 | Builder Pattern

### 這篇文章主要在講什麼

Builder Pattern 把複雜物件的建立步驟拆開，讓 client 不需要知道所有組裝細節。原文透過套餐、步驟式組裝來說明物件建立流程。

### 為什麼需要這個概念

當建構子參數很多、順序容易搞錯、物件組裝有多個可選步驟時，直接用 constructor 會很難讀。Builder 讓建立流程有語意，也能集中檢查必要資料。

### 核心重點

- Builder 適合複雜物件，不是所有 DTO 都需要。
- Director 可負責固定組裝流程，但很多 C# 專案會省略 Director。
- Builder 可以讓程式更像描述需求，而不是塞參數。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Builder | MealBuilder | 定義建立步驟 |
| ConcreteBuilder | SetMealBuilder | 實作實際組裝 |
| Product | Meal | 被建立的複雜物件 |
| Director | MealDirector | 控制組裝順序 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public sealed record OrderReport(string Title, DateOnly From, DateOnly To, bool IncludeDetails);

public sealed class OrderReportBuilder // Builder：逐步組裝報表需求
{
    private string _title = "訂單報表";
    private DateOnly _from;
    private DateOnly _to;
    private bool _includeDetails;

    public OrderReportBuilder ForRange(DateOnly from, DateOnly to)
    {
        _from = from;
        _to = to;
        return this;
    }

    public OrderReportBuilder WithDetails()
    {
        _includeDetails = true;
        return this;
    }

    public OrderReport Build() // Product：OrderReport
    {
        if (_from > _to) throw new InvalidOperationException("起日不可晚於迄日");
        return new OrderReport(_title, _from, _to, _includeDetails);
    }
}

OrderReport report = new OrderReportBuilder()
    .ForRange(new DateOnly(2026, 5, 1), new DateOnly(2026, 5, 31))
    .WithDetails()
    .Build();
```

### 程式碼對應概念說明

`OrderReportBuilder` 把「設定日期、是否含明細、最後驗證」拆成可讀步驟。呼叫端看得懂這份報表如何被建立。

### 負面例子 / 錯誤用法

對只有兩個欄位的物件做 Builder，會讓程式比原本更難懂。修正方向是：只有當建立流程複雜、可選項很多、驗證邏輯集中時才使用。

### Junior 常見誤解

Builder 不是為了少寫 constructor，而是為了讓複雜建立流程更安全、更有語意。

### 一句話總結

Builder 把複雜物件的建立流程變成清楚的步驟。

---

## Day 14：原型模式 | Prototype Pattern

### 這篇文章主要在講什麼

Prototype Pattern 透過複製既有物件來建立新物件。原文重點包含 clone、淺拷貝與深拷貝的差異。

### 為什麼需要這個概念

有些物件建立成本高，或初始化狀態很複雜。與其每次重新建立，不如複製一個已經準備好的 prototype，再調整少量欄位。

### 核心重點

- 淺拷貝只複製第一層，參考型欄位仍共用。
- 深拷貝會複製內部物件，避免彼此互相影響。
- Prototype 適合範本、表單預設值、遊戲物件、報表設定。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Prototype | Cloneable 物件 | 定義複製能力 |
| ConcretePrototype | 具體可複製物件 | 實作複製 |
| Client | 使用端 | 複製 prototype 後微調 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public sealed record ExportColumn(string Name);

public sealed class ExportTemplate // Prototype：可被複製的匯出範本
{
    public string Name { get; init; } = "";
    public List<ExportColumn> Columns { get; init; } = [];

    public ExportTemplate DeepCopy() => new()
    {
        Name = Name,
        Columns = Columns.Select(c => new ExportColumn(c.Name)).ToList()
    };
}

var monthlyTemplate = new ExportTemplate
{
    Name = "月報",
    Columns = [new ExportColumn("OrderId"), new ExportColumn("Amount")]
};

ExportTemplate custom = monthlyTemplate.DeepCopy();
custom.Columns.Add(new ExportColumn("CustomerName"));
```

### 程式碼對應概念說明

`monthlyTemplate` 是 prototype。`DeepCopy()` 建立獨立的新範本，`custom.Columns` 增加欄位時不會污染原始月報範本。

### 負面例子 / 錯誤用法

只用 `MemberwiseClone()` 做淺拷貝，內部 `List` 仍共用，改了副本會影響原物件。修正方向是針對可變參考物件做深拷貝，或把物件設計成 immutable。

### Junior 常見誤解

複製物件不是單純「看起來一樣」。你要確認內部參考是否也需要獨立。

### 一句話總結

Prototype 適合從既有範本複製新物件，但要小心淺拷貝共享狀態。

---

## Day 15：適配器模式 | Adapter Pattern

### 這篇文章主要在講什麼

Adapter Pattern 把不相容的介面轉成 client 期待的介面。原文用轉接頭概念說明：東西本身能用，但接不上，所以需要中間轉接。

### 為什麼需要這個概念

系統常會接第三方 API、舊系統或不同格式的資料。Adapter 讓既有 client 不必理解外部介面的怪異細節。

### 核心重點

- Adapter 解的是介面不相容，不是功能不存在。
- Adapter 可以包裝 legacy code，降低改動風險。
- 它常出現在第三方 SDK、支付、物流、簡訊、檔案格式轉換。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Target | Client 期待的介面 | 系統內部統一使用 |
| Adapter | 轉接器 | 把 Adaptee 轉成 Target |
| Adaptee | 被轉接者 | 既有或外部介面 |
| Client | 使用端 | 只依賴 Target |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public interface IPaymentGateway // Target：系統內期待的付款介面
{
    Task PayAsync(decimal amount);
}

public sealed class LegacyBankClient // Adaptee：舊銀行 SDK
{
    public void SendPayment(int cents) => Console.WriteLine($"pay {cents} cents");
}

public sealed class LegacyBankAdapter : IPaymentGateway // Adapter：轉接舊 SDK
{
    private readonly LegacyBankClient _client;
    public LegacyBankAdapter(LegacyBankClient client) => _client = client;

    public Task PayAsync(decimal amount)
    {
        _client.SendPayment((int)(amount * 100));
        return Task.CompletedTask;
    }
}

public sealed class CheckoutService
{
    public Task CheckoutAsync(IPaymentGateway gateway) => gateway.PayAsync(399);
}
```

### 程式碼對應概念說明

`CheckoutService` 只看 `IPaymentGateway`。舊銀行 SDK 需要 cents，但系統用 decimal 金額，轉換細節被 `LegacyBankAdapter` 隔離。

### 負面例子 / 錯誤用法

不要讓全系統到處直接呼叫 `LegacyBankClient`，否則外部 SDK 一改，整個系統都要跟著改。修正方向是把外部介面關在 Adapter 後面。

### Junior 常見誤解

Adapter 不是拿來「新增功能」的主角，它主要是讓不相容的東西能被既有系統使用。

### 一句話總結

Adapter 是系統和外部介面之間的轉接頭。

---

## Day 16：橋接模式 | Bridge Pattern

### 這篇文章主要在講什麼

Bridge Pattern 把抽象與實作拆開，讓兩邊可以獨立變化。原文用不同維度的變化說明：不要讓類別因多個維度組合而爆炸。

### 為什麼需要這個概念

如果一個功能同時有「形狀」與「顏色」兩個變動維度，繼承可能會產生 `RedCircle`、`BlueCircle`、`RedSquare`、`BlueSquare`。Bridge 把兩個維度拆成組合。

### 核心重點

- Bridge 用組合取代多層繼承。
- Abstraction 持有 Implementor。
- 適合功能與平台、報表與輸出格式、通知內容與發送管道分開變動。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Abstraction | 抽象端 | 定義高層操作 |
| RefinedAbstraction | 擴充抽象端 | 高層操作的變體 |
| Implementor | 實作端介面 | 定義底層能力 |
| ConcreteImplementor | 實體實作 | 實際執行底層行為 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public interface IMessageSender // Implementor：發送管道
{
    Task SendAsync(string text);
}

public sealed class EmailSender : IMessageSender // ConcreteImplementor
{
    public Task SendAsync(string text)
    {
        Console.WriteLine($"Email: {text}");
        return Task.CompletedTask;
    }
}

public abstract class Notification // Abstraction：通知類型
{
    protected readonly IMessageSender Sender;
    protected Notification(IMessageSender sender) => Sender = sender;
    public abstract Task NotifyAsync(string user);
}

public sealed class OrderPaidNotification : Notification // RefinedAbstraction
{
    public OrderPaidNotification(IMessageSender sender) : base(sender) { }
    public override Task NotifyAsync(string user) => Sender.SendAsync($"{user} 的訂單已付款");
}
```

### 程式碼對應概念說明

通知內容與發送管道是兩個變動維度。新增 SMS 不必新增一堆 `SmsOrderPaidNotification`，只要新增 `SmsSender`。

### 負面例子 / 錯誤用法

如果用繼承把每種通知與每種管道全部排列組合，類別數會快速增加。修正方向是把其中一個維度抽成介面，用組合連接。

### Junior 常見誤解

Bridge 和 Adapter 都有包裝感，但 Bridge 是設計初期就拆開變動維度；Adapter 通常是事後讓不相容介面接上。

### 一句話總結

Bridge 把兩個會變的維度拆開，避免繼承組合爆炸。

---

## Day 17：組合模式 | Composite Pattern

### 這篇文章主要在講什麼

Composite Pattern 讓單一物件與物件集合使用相同介面。原文用樹狀結構說明：葉節點與容器節點都可以被 client 一致處理。

### 為什麼需要這個概念

當資料天然是階層，例如資料夾、組織、選單、商品分類，client 不應該一直判斷「這是單一項目還是群組」。Composite 讓兩者有共同抽象。

### 核心重點

- Component 是共同介面。
- Leaf 是沒有子節點的物件。
- Composite 是可包含子節點的物件。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Component | 抽象構件 | 定義共同操作 |
| Leaf | 葉節點 | 單一物件 |
| Composite | 容器節點 | 管理子節點 |
| Client | 使用端 | 對 Component 操作 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public interface IMenuNode // Component：選單節點
{
    void Render(int depth = 0);
}

public sealed class MenuItem : IMenuNode // Leaf：單一選單項目
{
    private readonly string _name;
    public MenuItem(string name) => _name = name;
    public void Render(int depth = 0) => Console.WriteLine($"{new string(' ', depth)}- {_name}");
}

public sealed class MenuGroup : IMenuNode // Composite：可包含子節點
{
    private readonly string _name;
    private readonly List<IMenuNode> _children = [];
    public MenuGroup(string name) => _name = name;
    public void Add(IMenuNode node) => _children.Add(node);
    public void Render(int depth = 0)
    {
        Console.WriteLine($"{new string(' ', depth)}+ {_name}");
        foreach (var child in _children) child.Render(depth + 2);
    }
}
```

### 程式碼對應概念說明

`MenuGroup` 和 `MenuItem` 都能 `Render()`。Client 不需要知道目前節點是群組還是葉節點。

### 負面例子 / 錯誤用法

若所有操作都不是共同的，硬塞到同一個 Component 會讓 Leaf 出現一堆不支援的方法。修正方向是只抽共同能力，或把管理子節點的方法保留在 Composite。

### Junior 常見誤解

Composite 不是所有父子關係都要用；重點是 client 是否需要一致操作單一物件與集合。

### 一句話總結

Composite 讓樹狀結構中的單一物件與群組可以用同一套介面處理。

---

## Day 18：裝飾者模式 | Decorator Pattern

### 這篇文章主要在講什麼

Decorator Pattern 在不修改原物件的情況下，動態替物件增加行為。原文強調它比繼承更彈性，因為可以一層一層包裝。

### 為什麼需要這個概念

如果需求是「原功能加上快取、記錄、權限、重試」，用繼承會產生很多子類別。Decorator 可以在 runtime 組合功能。

### 核心重點

- Decorator 和被裝飾物件實作同一介面。
- Decorator 內部持有原物件。
- 多個 Decorator 可以疊加。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Component | 抽象元件 | 定義共同操作 |
| ConcreteComponent | 具體元件 | 原本功能 |
| Decorator | 抽象裝飾者 | 包裝 Component |
| ConcreteDecorator | 具體裝飾者 | 增加額外行為 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public interface IOrderRepository // Component：訂單查詢
{
    Task<string> FindAsync(int id);
}

public sealed class SqlOrderRepository : IOrderRepository // ConcreteComponent
{
    public Task<string> FindAsync(int id) => Task.FromResult($"Order {id}");
}

public sealed class LoggingOrderRepository : IOrderRepository // ConcreteDecorator
{
    private readonly IOrderRepository _inner;
    public LoggingOrderRepository(IOrderRepository inner) => _inner = inner;

    public async Task<string> FindAsync(int id)
    {
        Console.WriteLine($"query order {id}");
        return await _inner.FindAsync(id);
    }
}
```

### 程式碼對應概念說明

`LoggingOrderRepository` 不改 `SqlOrderRepository`，但能在查詢前增加 log。呼叫端仍然只依賴 `IOrderRepository`。

### 負面例子 / 錯誤用法

Decorator 疊太多層會讓除錯困難，不知道行為在哪一層發生。修正方向是保持裝飾者責任單一，並在 DI 註冊中清楚命名順序。

### Junior 常見誤解

Decorator 不是為了改變物件身分，而是替同一介面的物件增加前後處理。

### 一句話總結

Decorator 用包裝方式替物件增加功能，比繼承更容易組合。

---

## Day 19：外觀模式 | Facade Pattern

### 這篇文章主要在講什麼

Facade Pattern 對外提供簡化介面，隱藏子系統複雜流程。原文的重點是 client 不需要知道背後很多類別如何協作。

### 為什麼需要這個概念

一個操作常常牽涉多個 service：驗證、庫存、付款、通知、紀錄。Facade 把常用流程包成一個入口，降低 client 使用難度。

### 核心重點

- Facade 是簡化入口，不是把所有邏輯都塞成上帝類別。
- 子系統仍可被進階 client 直接使用。
- 適合封裝常見工作流程。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Facade | 外觀類別 | 提供簡化入口 |
| Subsystem | 子系統類別 | 實際處理各自工作 |
| Client | 使用端 | 呼叫 Facade |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public sealed class InventoryService { public void Reserve(int orderId) => Console.WriteLine("reserve"); }
public sealed class PaymentService { public void Charge(int orderId) => Console.WriteLine("charge"); }
public sealed class MailService { public void SendReceipt(int orderId) => Console.WriteLine("mail"); }

public sealed class CheckoutFacade // Facade：結帳流程入口
{
    private readonly InventoryService _inventory = new();
    private readonly PaymentService _payment = new();
    private readonly MailService _mail = new();

    public void Checkout(int orderId)
    {
        _inventory.Reserve(orderId);
        _payment.Charge(orderId);
        _mail.SendReceipt(orderId);
    }
}

new CheckoutFacade().Checkout(1001);
```

### 程式碼對應概念說明

Client 只呼叫 `Checkout()`，不需要知道結帳背後要先保留庫存、再付款、再寄信。

### 負面例子 / 錯誤用法

Facade 若開始處理所有商業規則，很容易變成過大的 service。修正方向是 Facade 只編排流程，真正邏輯留在子系統。

### Junior 常見誤解

Facade 不是取代子系統，而是給常用情境一個簡單入口。

### 一句話總結

Facade 把複雜子系統包成簡單好用的入口。

---

## Day 20：享元模式 | Flyweight Pattern

### 這篇文章主要在講什麼

Flyweight Pattern 透過共享可重複使用的物件，降低大量物件造成的記憶體成本。原文重點是區分可共享的內部狀態與不可共享的外部狀態。

### 為什麼需要這個概念

當系統建立大量相似物件，例如棋子、字元、地圖圖示、商品規格，如果每個都存完整資料會浪費記憶體。Flyweight 把共用資料抽出共享。

### 核心重點

- Intrinsic state 是可共享狀態。
- Extrinsic state 是使用時由外部傳入的狀態。
- Factory 通常負責快取與重用 flyweight。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Flyweight | 享元介面 | 定義共享物件行為 |
| ConcreteFlyweight | 具體享元 | 保存可共享狀態 |
| FlyweightFactory | 享元工廠 | 快取與重用物件 |
| Client | 使用端 | 傳入外部狀態 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public sealed class ProductIcon // ConcreteFlyweight：共享圖示樣式
{
    public ProductIcon(string name, string color)
    {
        Name = name;
        Color = color;
    }

    public string Name { get; }
    public string Color { get; }
    public void Draw(int x, int y) => Console.WriteLine($"{Name}-{Color} at {x},{y}");
}

public sealed class ProductIconFactory // FlyweightFactory
{
    private readonly Dictionary<string, ProductIcon> _cache = [];

    public ProductIcon Get(string name, string color)
    {
        string key = $"{name}:{color}";
        return _cache.TryGetValue(key, out var icon)
            ? icon
            : _cache[key] = new ProductIcon(name, color);
    }
}

var factory = new ProductIconFactory();
factory.Get("sale", "red").Draw(10, 20);
factory.Get("sale", "red").Draw(80, 40);
```

### 程式碼對應概念說明

`name` 與 `color` 是可共享狀態；座標 `x, y` 是外部狀態，每次繪製時傳入。

### 負面例子 / 錯誤用法

把會變的使用者資料也放進共享物件，會造成不同使用者互相污染。修正方向是只共享不變或可安全共用的狀態。

### Junior 常見誤解

Flyweight 不是單純 cache。它特別強調把狀態拆成可共享與不可共享兩部分。

### 一句話總結

Flyweight 用共享物件節省大量相似物件的記憶體成本。

---

## Day 21：代理模式 | Proxy Pattern

### 這篇文章主要在講什麼

Proxy Pattern 讓代理物件控制對真實物件的存取。原文重點是 client 透過代理間接操作目標物件。

### 為什麼需要這個概念

你可能需要在真正呼叫前做權限檢查、延遲載入、快取、遠端呼叫或紀錄。Proxy 讓這些控制邏輯放在代理層，而不是污染真實物件。

### 核心重點

- Proxy 和 RealSubject 實作同一介面。
- Proxy 可以在呼叫前後增加控制。
- 常見類型包含 protection proxy、virtual proxy、remote proxy。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Subject | 抽象主題 | 定義共同介面 |
| RealSubject | 真實主題 | 實際執行工作 |
| Proxy | 代理 | 控制對真實主題的存取 |
| Client | 使用端 | 呼叫 Subject |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public interface IReportExporter // Subject
{
    Task ExportAsync(string userRole);
}

public sealed class CsvReportExporter : IReportExporter // RealSubject
{
    public Task ExportAsync(string userRole)
    {
        Console.WriteLine("export csv");
        return Task.CompletedTask;
    }
}

public sealed class PermissionReportExporterProxy : IReportExporter // Proxy
{
    private readonly IReportExporter _inner;
    public PermissionReportExporterProxy(IReportExporter inner) => _inner = inner;

    public Task ExportAsync(string userRole)
    {
        if (userRole != "Admin") throw new UnauthorizedAccessException();
        return _inner.ExportAsync(userRole);
    }
}
```

### 程式碼對應概念說明

`CsvReportExporter` 只負責匯出；權限檢查放在 Proxy。Client 仍只看到 `IReportExporter`。

### 負面例子 / 錯誤用法

如果 Proxy 做了太多商業邏輯，真實物件與代理的責任會混亂。修正方向是 Proxy 只做存取控制、延遲、快取、遠端轉發等橫切邏輯。

### Junior 常見誤解

Proxy 和 Decorator 結構很像；Proxy 重點是控制存取，Decorator 重點是擴充功能。

### 一句話總結

Proxy 是真實物件前面的存取控制層。

---

## Day 22：責任鏈模式 | Chain of Responsibility Pattern

### 這篇文章主要在講什麼

Chain of Responsibility 把請求沿著一串 handler 傳遞，直到某個 handler 處理它，或每個 handler 都做一段處理。

### 為什麼需要這個概念

如果一個請求需要經過多個規則，例如審核、驗證、折扣判斷，用一大串 if-else 會難維護。責任鏈讓每個節點只管自己的條件。

### 核心重點

- Handler 定義處理與傳遞下一棒。
- ConcreteHandler 處理自己能處理的請求。
- 適合審核流程、middleware、驗證管線。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Handler | 抽象處理者 | 定義處理介面與下一個處理者 |
| ConcreteHandler | 實體處理者 | 判斷是否處理或往後傳 |
| Client | 使用端 | 組裝責任鏈並送出請求 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public sealed record ExpenseRequest(decimal Amount);

public abstract class Approver // Handler：審核者
{
    private Approver? _next;
    public Approver SetNext(Approver next) { _next = next; return next; }

    public virtual void Handle(ExpenseRequest request)
    {
        _next?.Handle(request);
    }
}

public sealed class ManagerApprover : Approver // ConcreteHandler
{
    public override void Handle(ExpenseRequest request)
    {
        if (request.Amount <= 10_000) Console.WriteLine("主管核准");
        else base.Handle(request);
    }
}

public sealed class DirectorApprover : Approver // ConcreteHandler
{
    public override void Handle(ExpenseRequest request) => Console.WriteLine("處長核准");
}

var manager = new ManagerApprover();
manager.SetNext(new DirectorApprover());
manager.Handle(new ExpenseRequest(15_000));
```

### 程式碼對應概念說明

`ManagerApprover` 處理小額，超過就交給下一個 handler。Client 只把請求丟給鏈頭。

### 負面例子 / 錯誤用法

鏈太長且沒有清楚結束條件，請求可能被吞掉或難追蹤。修正方向是設計預設 handler，並加上 log 或結果回傳。

### Junior 常見誤解

責任鏈不是一定只會有一個 handler 處理；有些管線式實作會讓每個 handler 都處理一部分。

### 一句話總結

責任鏈把一大串判斷拆成可排列的處理節點。

---

## Day 23：命令模式 | Command Pattern

### 這篇文章主要在講什麼

Command Pattern 把「請求」封裝成物件，讓發出命令的人與真正執行命令的人分離。

### 為什麼需要這個概念

當你需要延遲執行、排程、重試、撤銷、記錄操作歷史時，把動作封裝成 command 會比直接呼叫 method 更有彈性。

### 核心重點

- Command 代表一個可執行請求。
- Receiver 真正做事。
- Invoker 持有並觸發 command。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Command | 抽象命令 | 定義執行方法 |
| ConcreteCommand | 實體命令 | 綁定 Receiver 與操作 |
| Receiver | 接收者 | 真正執行工作 |
| Invoker | 呼叫者 | 觸發命令 |
| Client | 使用端 | 建立與組裝命令 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public interface ICommand // Command
{
    Task ExecuteAsync();
}

public sealed class EmailService // Receiver
{
    public Task SendAsync(string email) => Task.Run(() => Console.WriteLine($"send {email}"));
}

public sealed class SendWelcomeEmailCommand : ICommand // ConcreteCommand
{
    private readonly EmailService _service;
    private readonly string _email;
    public SendWelcomeEmailCommand(EmailService service, string email)
    {
        _service = service;
        _email = email;
    }

    public Task ExecuteAsync() => _service.SendAsync(_email);
}

public sealed class JobQueue // Invoker
{
    public Task EnqueueAndRunAsync(ICommand command) => command.ExecuteAsync();
}
```

### 程式碼對應概念說明

`JobQueue` 不知道歡迎信怎麼寄，只知道 command 可以執行。寄信細節留給 `EmailService`。

### 負面例子 / 錯誤用法

每個簡單 method 都包成 command，會讓類別數暴增。修正方向是只在需要排程、撤銷、重試、紀錄或解耦呼叫端時使用。

### Junior 常見誤解

Command 不是單純 service method 的換皮；它的價值在於把請求變成可保存、傳遞、排程的物件。

### 一句話總結

Command 把動作封裝成物件，讓請求可以被排程、記錄或延遲執行。

---

## Day 24：迭代器模式 | Iterator Pattern

### 這篇文章主要在講什麼

Iterator Pattern 提供一致方式走訪集合，而不暴露集合內部結構。原文重點是讓不同集合能用同一種方式遍歷。

### 為什麼需要這個概念

Client 不應該知道集合是陣列、樹、鏈結串列還是資料庫游標。Iterator 把走訪方式包起來。

### 核心重點

- Iterator 定義取得下一個元素的方式。
- Aggregate 提供取得 iterator 的方法。
- C# 的 `IEnumerable<T>` 與 `IEnumerator<T>` 就是常見迭代器概念。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Iterator | 抽象迭代器 | 定義走訪介面 |
| ConcreteIterator | 實體迭代器 | 實作走訪 |
| Aggregate | 聚合物件 | 提供 iterator |
| Client | 使用端 | 使用 iterator 遍歷 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public sealed class RecentOrderCollection // Aggregate：訂單集合
{
    private readonly List<string> _orders = ["A001", "A002", "A003"];

    public IEnumerable<string> GetRecentOrders() // Iterator：yield 產生走訪方式
    {
        for (int i = _orders.Count - 1; i >= 0; i--)
        {
            yield return _orders[i];
        }
    }
}

foreach (string orderNo in new RecentOrderCollection().GetRecentOrders())
{
    Console.WriteLine(orderNo);
}
```

### 程式碼對應概念說明

使用端只用 `foreach`，不需要知道內部是反向走訪或資料如何存放。

### 負面例子 / 錯誤用法

遍歷時同時修改集合，容易造成錯誤或漏資料。修正方向是先複製快照，或明確設計可修改的迭代流程。

### Junior 常見誤解

Iterator 不是只有手寫類別才算；在 C# 中 `foreach` 背後就是迭代器思維。

### 一句話總結

Iterator 隱藏集合內部結構，提供一致的走訪方式。

---

## Day 25：中介者模式 | Mediator Pattern

### 這篇文章主要在講什麼

Mediator Pattern 用一個中介者處理多個物件之間的互動。原文用產品經理協調工程、設計、行銷、客服團隊說明。

### 為什麼需要這個概念

如果每個團隊都直接找其他團隊，關係會變成網狀，耦合很高。Mediator 把溝通集中到中介者，讓同事物件不必彼此知道。

### 核心重點

- Mediator 管理同事物件之間的互動。
- Colleague 只認識 Mediator。
- 中介者過大時會變成新的複雜中心。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Mediator | Mediator | 定義註冊與轉發 |
| ConcreteMediator | ProductManager | 管理團隊與訊息轉發 |
| Colleague | Colleague | 團隊抽象 |
| ConcreteColleague | EngineeringTeam, DesignTeam, MarketingTeam, CustomerService | 實際團隊 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public interface ITeamMediator // Mediator：團隊訊息中介
{
    void Register(string name, ITeam team);
    void Send(string to, string message);
}

public interface ITeam // Colleague：團隊
{
    void Receive(string message);
}

public sealed class ProductManagerMediator : ITeamMediator // ConcreteMediator
{
    private readonly Dictionary<string, ITeam> _teams = [];
    public void Register(string name, ITeam team) => _teams[name] = team;
    public void Send(string to, string message) => _teams[to].Receive(message);
}

public sealed class EngineeringTeam : ITeam // ConcreteColleague
{
    public void Receive(string message) => Console.WriteLine($"工程收到：{message}");
}
```

### 程式碼對應概念說明

團隊不直接持有彼此，只透過 `ProductManagerMediator` 轉送訊息。

### 負面例子 / 錯誤用法

所有跨團隊邏輯都塞進 Mediator，會形成超大協調者。修正方向是讓 Mediator 只負責溝通路由，複雜規則拆到 domain service。

### Junior 常見誤解

Mediator 不是「萬能管理者」。它降低物件彼此依賴，但也可能把複雜度集中到自己身上。

### 一句話總結

Mediator 把多對多溝通改成透過中介者協調。

---

## Day 26：狀態模式 | State Pattern

### 這篇文章主要在講什麼

State Pattern 讓物件在不同狀態下有不同表現，並把狀態相關行為封裝成獨立狀態類別。

### 為什麼需要這個概念

當一個物件有很多狀態，例如訂單的待付款、已付款、已取消，如果全部用 if-else 判斷，狀態轉換會散落各處。State 把各狀態行為集中。

### 核心重點

- Context 保存目前狀態。
- State 定義狀態行為。
- ConcreteState 實作特定狀態下的行為與轉換。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Context | 環境類別 | 保存目前狀態 |
| State | 抽象狀態 | 定義狀態行為 |
| ConcreteState | 實體狀態 | 實作特定狀態行為 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public interface IOrderState // State：訂單狀態
{
    void Pay(OrderContext order);
    void Cancel(OrderContext order);
}

public sealed class PendingPaymentState : IOrderState // ConcreteState
{
    public void Pay(OrderContext order) => order.ChangeState(new PaidState());
    public void Cancel(OrderContext order) => order.ChangeState(new CancelledState());
}

public sealed class PaidState : IOrderState
{
    public void Pay(OrderContext order) => throw new InvalidOperationException("已付款");
    public void Cancel(OrderContext order) => throw new InvalidOperationException("已付款不可取消");
}

public sealed class CancelledState : IOrderState
{
    public void Pay(OrderContext order) => throw new InvalidOperationException("已取消");
    public void Cancel(OrderContext order) => throw new InvalidOperationException("已取消");
}

public sealed class OrderContext // Context：保存目前狀態
{
    private IOrderState _state = new PendingPaymentState();
    public void ChangeState(IOrderState state) => _state = state;
    public void Pay() => _state.Pay(this);
    public void Cancel() => _state.Cancel(this);
}
```

### 程式碼對應概念說明

`OrderContext` 不自己寫一堆狀態判斷，而是把行為委派給目前的 `IOrderState`。

### 負面例子 / 錯誤用法

狀態很少且規則簡單時，State Pattern 會讓類別變多。修正方向是先用清楚的 enum 與 switch，等狀態行為膨脹再重構。

### Junior 常見誤解

State 和 Strategy 都會委派行為；State 通常會因內部狀態轉換而改變下一次行為。

### 一句話總結

State 把不同狀態下的行為封裝起來，避免狀態判斷散落各處。

---

## Day 27：備忘錄模式 | Memento Pattern

### 這篇文章主要在講什麼

Memento Pattern 保存物件某一刻的狀態，之後可以恢復。原文重點是保存狀態但不破壞封裝。

### 為什麼需要這個概念

常見場景是 undo、草稿保存、遊戲存檔、編輯器快照。你需要能回復狀態，但不想讓外部直接改物件內部欄位。

### 核心重點

- Originator 建立與恢復 memento。
- Memento 保存狀態。
- Caretaker 管理 memento，但不理解內容。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Originator | 發起人 | 建立與還原狀態 |
| Memento | 備忘錄 | 保存狀態 |
| Caretaker | 管理者 | 管理備忘錄歷史 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public sealed record EditorMemento(string Content); // Memento：保存狀態

public sealed class TextEditor // Originator：文字編輯器
{
    public string Content { get; private set; } = "";
    public void Type(string text) => Content += text;
    public EditorMemento Save() => new(Content);
    public void Restore(EditorMemento memento) => Content = memento.Content;
}

public sealed class UndoHistory // Caretaker：管理歷史
{
    private readonly Stack<EditorMemento> _history = [];
    public void Push(EditorMemento memento) => _history.Push(memento);
    public EditorMemento Pop() => _history.Pop();
}

var editor = new TextEditor();
var history = new UndoHistory();
history.Push(editor.Save());
editor.Type("hello");
editor.Restore(history.Pop());
```

### 程式碼對應概念說明

`UndoHistory` 只保存 `EditorMemento`，不直接修改 `TextEditor.Content`。真正恢復狀態由 `TextEditor` 自己做。

### 負面例子 / 錯誤用法

把完整大物件每次都深拷貝保存，可能造成大量記憶體消耗。修正方向是只保存必要狀態，或使用差異紀錄。

### Junior 常見誤解

Memento 不是資料庫備份；它通常是應用程式內部短期狀態恢復機制。

### 一句話總結

Memento 在不破壞封裝的前提下保存與恢復物件狀態。

---

## Day 28：觀察者模式 | Observer Pattern

### 這篇文章主要在講什麼

Observer Pattern 描述一對多依賴：Subject 狀態改變時，通知所有 Observer。原文用 YouTuber 發片通知粉絲與廠商來說明。

### 為什麼需要這個概念

當一個事件發生後有多個反應方，例如訂單付款後寄信、加點數、通知倉庫，不應該讓訂單物件直接知道所有後續行為細節。

### 核心重點

- Subject 管理觀察者清單。
- Observer 定義收到通知後的反應。
- 觀察者多時要注意效能、例外處理與非同步。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Subject | YoutuberPeeta | 保存與通知觀察者 |
| ConcreteSubject | YoutuberPeeta | 發布影片事件 |
| Observer | Fans, Vendor 的抽象 | 定義接收通知 |
| ConcreteObserver | Fans, Vendor | 依事件做反應 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public sealed record OrderPaid(int OrderId);

public interface IOrderPaidObserver // Observer：訂單付款觀察者
{
    Task HandleAsync(OrderPaid message);
}

public sealed class MailObserver : IOrderPaidObserver // ConcreteObserver
{
    public Task HandleAsync(OrderPaid message)
    {
        Console.WriteLine($"寄送訂單 {message.OrderId} 收據");
        return Task.CompletedTask;
    }
}

public sealed class OrderSubject // Subject：管理通知
{
    private readonly List<IOrderPaidObserver> _observers = [];
    public void Subscribe(IOrderPaidObserver observer) => _observers.Add(observer);

    public Task PublishAsync(OrderPaid message) =>
        Task.WhenAll(_observers.Select(o => o.HandleAsync(message)));
}
```

### 程式碼對應概念說明

`OrderSubject` 不知道每個觀察者的細節，只負責通知。新增點數觀察者時，不必修改付款發布流程。

### 負面例子 / 錯誤用法

觀察者中任一個失敗導致整個流程中斷，會讓事件系統不穩。修正方向是設計例外隔離、重試或訊息佇列。

### Junior 常見誤解

Observer 不是一定要同步呼叫。在實務上，事件通知常會做成 async、message bus 或 background job。

### 一句話總結

Observer 讓一個事件發生後，多個觀察者能被通知並各自反應。

---

## Day 29：策略模式 | Strategy Pattern

### 這篇文章主要在講什麼

Strategy Pattern 定義一系列可替換演算法，讓使用端在不改流程的情況下切換策略。原文用出遊交通方式與剪刀石頭布說明。

### 為什麼需要這個概念

當一個流程中只有某段演算法會變，例如折扣、運費、排序、稅率，使用大量 if-else 會違反開閉原則。Strategy 把變動演算法拆成獨立類別。

### 核心重點

- Strategy 定義演算法介面。
- ConcreteStrategy 實作不同演算法。
- Context 使用策略，但不關心策略內部細節。

### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Strategy | Strategy | 定義策略方法 |
| ConcreteStrategy | Paper, Scissors, Stone | 實作具體策略 |
| Context | Context | 持有並執行目前策略 |

### C# 程式碼例子

```csharp
// 範例用途：示範「C# 程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
public interface IDiscountStrategy // Strategy：折扣策略
{
    decimal Calculate(decimal amount);
}

public sealed class VipDiscountStrategy : IDiscountStrategy // ConcreteStrategy
{
    public decimal Calculate(decimal amount) => amount * 0.8m;
}

public sealed class NoDiscountStrategy : IDiscountStrategy // ConcreteStrategy
{
    public decimal Calculate(decimal amount) => amount;
}

public sealed class CheckoutContext // Context：結帳流程
{
    private readonly IDiscountStrategy _strategy;
    public CheckoutContext(IDiscountStrategy strategy) => _strategy = strategy;
    public decimal GetPayableAmount(decimal amount) => _strategy.Calculate(amount);
}

var checkout = new CheckoutContext(new VipDiscountStrategy());
Console.WriteLine(checkout.GetPayableAmount(1000));
```

### 程式碼對應概念說明

`CheckoutContext` 不寫 VIP、一般會員、活動會員的 if-else，而是把折扣演算法交給策略物件。

### 負面例子 / 錯誤用法

如果 client 仍然用巨大 switch 決定每個策略細節，策略模式只把 if-else 搬家。修正方向是用 factory、DI keyed service 或設定檔集中選擇策略。

### Junior 常見誤解

Strategy 和 State 長得像，但 Strategy 通常由外部選擇演算法；State 通常由物件內部狀態轉換驅動。

### 一句話總結

Strategy 把可替換演算法抽出，讓流程穩定、演算法可擴充。

---

## 跨文章比較與學習建議

| 容易混淆 | 差異 |
| --- | --- |
| Factory Method vs Abstract Factory | Factory Method 建立單一產品；Abstract Factory 建立一整組相容產品 |
| Adapter vs Bridge | Adapter 多是事後轉接不相容介面；Bridge 是事前拆開兩個變動維度 |
| Decorator vs Proxy | Decorator 加功能；Proxy 控制存取 |
| State vs Strategy | State 依內部狀態改變行為；Strategy 由外部選擇演算法 |
| Facade vs Mediator | Facade 簡化子系統入口；Mediator 協調多個物件互動 |

## Junior Developer 建議

先不要急著把所有 pattern 都套進專案。比較好的練習方式是：先寫出直覺版本，再觀察哪裡開始出現重複、if-else 膨脹、建立流程複雜、物件互相知道太多細節。當痛點真的出現，再回頭套用對應 pattern，你會更清楚 pattern 解決的是什麼問題。

設計模式不是讓程式看起來比較高級，而是讓變動有地方可以放。
