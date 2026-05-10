# 使用 Claude Code：HTML 不合理的有效性整理

來源：

- X 文章：<https://x.com/trq212/status/2052809885763747935>
- 作者文中範例集：<https://thariqs.github.io/html-effectiveness/>
- 發布時間：2026-05-08

## 這篇文章主要在講什麼

這篇文章的核心主張是：在 Claude Code 這類 coding agent 工作流裡，Markdown 仍然很方便，但它不一定永遠是最適合的輸出格式。當 agent 產出的內容變成規格書、研究報告、PR 說明、設計探索、互動式編輯器或可視化文件時，HTML 往往比 Markdown 更容易閱讀、分享、互動與承載複雜資訊。

作者不是說 Markdown 沒用了，而是提醒我們：當你不是要自己手改文字，而是要讓 agent 產生一份「人真的會讀、會比較、會操作」的文件時，HTML 可能是更好的交付物。

## 為什麼需要這個概念

很多 junior 工程師使用 AI agent 時，會習慣要求它輸出 Markdown plan、Markdown spec 或 Markdown summary。這在小任務很好用，但任務一變大，問題會開始出現：

- 規格超過一百行後，讀者很容易只看標題與結論。
- ASCII diagram 能表達的資訊有限，複雜流程很快變難讀。
- code review、資料流、UI mockup、流程圖、風險分級這些東西，在純 Markdown 裡常常要靠大量文字補洞。
- 分享 Markdown 給非工程同事時，對方可能只看到原始文字，閱讀體驗不穩定。
- 如果你已經主要透過 prompt 修改文件，Markdown「好手動編輯」的優勢就沒有那麼重要。

所以這篇文章真正要教的是：不要只把 agent 當成文字產生器，也可以把它當成一次性工具、可視化文件與互動介面的產生器。

## 學完這篇你應該會做到什麼

讀完後，你應該能判斷：

- 什麼情境適合請 Claude Code 產生 HTML，而不是 Markdown。
- 如何把 HTML 用在規格探索、PR 說明、研究報告與設計 prototype。
- 怎麼要求 agent 產出可閱讀、可比較、可互動、可回拋 prompt 的 HTML artifact。
- HTML 輸出有哪些代價，例如生成較慢、版本控制 diff 較吵、需要基本設計品質控制。
- 在工作流程中怎麼驗證這份 HTML 是否真的比 Markdown 更有用。

## 核心重點

### 1. HTML 的資訊密度更高

Markdown 適合標題、段落、清單、code block 與簡單表格。HTML 除了這些，還可以搭配 CSS、SVG、Canvas、圖片、互動元件與 JavaScript。這代表它能把流程圖、差異標註、設計比較、可調參數、資料表與互動輸出放在同一份文件裡。

直覺理解：

- Markdown 像一份整理好的文字筆記。
- HTML 像一個可以被瀏覽器打開的小型閱讀介面。

當任務只需要純文字摘要，Markdown 很好。當任務需要「看懂關係、比較選項、操作參數、展示流程」，HTML 的表達能力就更強。

### 2. HTML 更適合大型 spec、plan 與研究報告

作者提到，agent 能做的工作越複雜，產出的 plan 和 spec 也越大。問題不是 agent 不會寫，而是人不一定會讀。HTML 可以用 tabs、區塊、顏色、圖表、流程圖、側邊導覽與 responsive layout 讓閱讀變得比較像瀏覽產品介面，而不是硬啃長文。

實務上，這特別適合：

- 大型 feature 的 implementation plan。
- 跨模組資料流說明。
- 架構探索與方案比較。
- 技術研究報告。
- incident report 或週報。

### 3. HTML 更容易分享

Markdown 在 GitHub 上很好讀，但離開 GitHub 或 IDE 後就不一定。HTML 只要放到 S3、GitHub Pages、內部靜態檔服務或直接寄檔案，對方用瀏覽器就能看。

這個差異很實務：你的規格、報告或 PR 說明如果更容易打開，就更有機會真的被同事讀完。

### 4. HTML 可以做雙向互動

HTML 不只是呈現結果，也能讓使用者操作。例如：

- 用 slider 調整動畫速度。
- 用拖拉卡片重新排序 Linear tickets。
- 用表單編輯 feature flag。
- 用 prompt editor 即時預覽不同變數填入後的結果。
- 最後用 copy button 匯出 JSON、Markdown、diff 或 prompt，再貼回 Claude Code。

這是 Markdown 很難做到的地方。HTML 可以變成「一次性工作台」，幫你把難以用文字描述的選擇變成可操作的介面。

### 5. Claude Code 的優勢是能吃進工作區上下文

作者特別強調，為什麼不是只用一般聊天介面產生 HTML，而是用 Claude Code。原因是 Claude Code 能讀你的檔案系統、程式碼、git history，也可能透過 MCP 讀 Slack、Linear 等來源。這些上下文可以被整理成一份更貼近真實專案的 HTML 文件。

換句話說，HTML 不是重點的全部；真正有價值的是「agent 讀了專案脈絡後，把複雜資訊重新包成可閱讀介面」。

## 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 輸出格式 | HTML artifact / HTML file | Agent 產生、用瀏覽器閱讀或操作的單頁文件 |
| 傳統輸出格式 | Markdown | 適合文字筆記、簡單規格與可手動編輯的文件 |
| 工作環境 | Claude Code | 能讀取程式碼、檔案與工具上下文的 coding agent |
| 使用者 | Human reviewer / developer | 閱讀、比較、操作、驗證 agent 產出的人 |
| 互動輸出 | Copy as JSON / copy as prompt / copy diff | 把使用者在 HTML 裡的操作結果匯出，讓 agent 繼續處理 |

## 實務使用情境

適合使用 HTML 的情境：

- 你要比較多個設計方案，而不是只讀一份文字建議。
- 你要 code review，想看 diff、註解、風險分級與流程圖。
- 你要理解一個陌生模組，希望看到資料流、關鍵程式片段與 gotchas。
- 你要做研究報告，內容來自 codebase、git history、Slack、Linear 或網路資料。
- 你要調 prompt、調 config、排 tickets、標註資料集，並把結果匯出。
- 你要把規格交給同事、主管或 reviewer，並提高對方真的打開閱讀的機率。

## 不適合使用的情境

不適合硬改成 HTML 的情境：

- 內容很短，只需要幾行結論或待辦事項。
- 文件需要長期用 git diff 精準 review。
- 團隊規範要求所有設計決策都要留在 Markdown、ADR 或 issue 裡。
- 你沒有時間等 agent 產生較完整的 HTML。
- HTML 只是裝飾，沒有提升閱讀、比較、操作或分享價值。
- 文件會被很多人直接手改，且沒有格式化或 review 機制。

## 真實工作流程例子

- 工作任務：你接到一個 code review 任務，PR 修改了訂單結帳流程，但你不熟悉付款狀態、庫存扣除與失敗回滾的資料流。你想請 Claude Code 產生一份 HTML code explainer，讓 reviewer 能快速看懂風險。
- 你先判斷：先判斷這份輸出是「協助 review 的閱讀介面」，不是正式產品頁面，也不是取代 PR 本身。重點應放在 diff、資料流、風險點、驗證方式，而不是做漂亮動畫。
- 會動到：檢查 PR diff、`OrderController`、`CheckoutService`、`PaymentGatewayClient`、`InventoryService`、測試檔、README 或 PR 說明草稿；新增一個暫時的 `checkout-review.html`。
- 資料怎麼流：reviewer 從 PR 入口讀到 HTML，HTML 顯示從 HTTP request 到 service、外部付款、庫存異動、DB 狀態與錯誤處理的流向，最後 reviewer 回到 PR 留下具體意見。
- 流程路線圖：

```text
PR diff -> Claude Code 讀取相關檔案 -> 產生 HTML review artifact
-> reviewer 用瀏覽器閱讀 -> 找出風險與測試缺口 -> 回到 PR 留 comment
```

- 工作中會寫 / 檢查的片段：

```text
請讀取這個 PR 的 diff 和 checkout 相關檔案，建立 checkout-review.html。

HTML 需要包含：
1. 一張從 POST /checkout 到付款、庫存、訂單狀態的資料流圖。
2. 實際 diff 的重點片段，旁邊用 inline annotation 說明風險。
3. 用紅 / 黃 / 綠標示 findings 嚴重度。
4. 一個「交付前測試清單」區塊，列出 happy path、付款失敗、庫存不足、重複送出。
5. 一個 copy button，可以複製 Markdown 格式的 review comment。
```

- 交付前驗證：
  - 用瀏覽器打開 `checkout-review.html`，確認資料流圖、diff 片段與測試清單都能看。
  - 回到 PR diff 抽查 3 個 HTML 提到的程式片段，確認沒有引用錯檔案或舊邏輯。
  - 檢查 copy button 匯出的 review comment 是否能直接貼到 GitHub PR。
  - 請另一位不熟 checkout 的同事快速讀 3 分鐘，看是否能講出主要風險。
- 常見卡點：junior 容易要求「幫我做漂亮 HTML」，但沒有說清楚閱讀任務。第一步應該先定義讀者是誰、要判斷什麼、看完要採取什麼行動。

## 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 來源資料 | PR diff、相關 service、測試檔、issue 需求 | 提供 agent 產生 HTML 的真實上下文 |
| 輸出文件 | `checkout-review.html` | 承載資料流圖、diff annotation、風險表與測試清單 |
| 使用端 | 瀏覽器、GitHub PR comment | reviewer 閱讀、複製結論並回到 PR 溝通 |
| 驗證方式 | 本機瀏覽、抽查程式碼、跑測試 | 確認 HTML 沒有看似漂亮但內容錯誤 |

## 完整實作流程、範例與注意事項

完整流程：

1. 先決定 HTML 的用途，例如 PR review、設計比較、研究報告或一次性資料編輯器。
2. 列出 agent 需要讀的資料來源，例如檔案、git diff、issue、log、測試結果。
3. 在 prompt 裡明確指定讀者、輸出結構與驗證目標。
4. 要求 HTML 包含可視化或互動，而不是把 Markdown 內容包進 `<div>`。
5. 用瀏覽器打開檔案，檢查資訊是否真的更容易讀。
6. 抽查 HTML 內引用的程式碼與結論，避免 agent 因為追求版面而產生錯誤摘要。
7. 如果要分享，放到團隊允許的位置，例如 GitHub Pages、S3、內部靜態站或 PR artifact。

範例 prompt：

```text
請建立一個 HTML artifact，幫助後端工程師 review 這個訂單結帳 PR。

讀者：
- 不熟悉 checkout 模組的 reviewer。

請讀取：
- git diff
- src/Orders/OrderController.cs
- src/Orders/CheckoutService.cs
- src/Payments/PaymentGatewayClient.cs
- tests/Orders/CheckoutServiceTests.cs

HTML 內容需要：
- 用 SVG 或 HTML/CSS 畫出資料流。
- 顯示 3 到 5 個最重要的 diff 片段，旁邊加註解。
- 用表格列出風險、影響、建議測試。
- 加一個 copy review summary 按鈕，輸出 Markdown。

請不要只做視覺裝飾。重點是讓 reviewer 更快判斷這個 PR 能不能 merge。
```

如果你想在 .NET 團隊中補上一個很小的 HTML artifact 產生流程，可以把它當成開發輔助輸出，不要放進正式 runtime：

```csharp
// 範例用途：示範如何在本機工具中輸出一份簡單 HTML review artifact。
// 主要輸入：
// - title：這份 review artifact 的標題。
// - findings：從 PR diff 或人工整理來的 review 重點。
// 回傳結果 / 副作用：
// - 在指定路徑寫出 HTML 檔案，讓 reviewer 用瀏覽器打開。
// 注意：這只是本機輔助工具，不代表正式產品功能。
public sealed record ReviewFinding(string Severity, string Area, string Summary);

public static class ReviewHtmlWriter
{
    public static async Task WriteAsync(
        string outputPath,
        string title,
        IReadOnlyList<ReviewFinding> findings)
    {
        string rows = string.Join(Environment.NewLine, findings.Select(f => $"""
            <tr>
              <td>{System.Net.WebUtility.HtmlEncode(f.Severity)}</td>
              <td>{System.Net.WebUtility.HtmlEncode(f.Area)}</td>
              <td>{System.Net.WebUtility.HtmlEncode(f.Summary)}</td>
            </tr>
            """));

        string html = $$"""
        <!doctype html>
        <html lang="zh-Hant">
        <head>
          <meta charset="utf-8">
          <title>{{System.Net.WebUtility.HtmlEncode(title)}}</title>
          <style>
            body { font-family: system-ui, sans-serif; margin: 32px; line-height: 1.6; }
            table { border-collapse: collapse; width: 100%; }
            th, td { border: 1px solid #d0d7de; padding: 8px; text-align: left; }
            th { background: #f6f8fa; }
          </style>
        </head>
        <body>
          <h1>{{System.Net.WebUtility.HtmlEncode(title)}}</h1>
          <p>這份檔案用來輔助 PR review，請回到原始 diff 抽查內容是否正確。</p>
          <table>
            <thead>
              <tr><th>嚴重度</th><th>區域</th><th>摘要</th></tr>
            </thead>
            <tbody>
              {{rows}}
            </tbody>
          </table>
        </body>
        </html>
        """;

        await File.WriteAllTextAsync(outputPath, html);
    }
}
```

操作方式：

1. 從 PR review 筆記或 agent 輸出整理出 `ReviewFinding`。
2. 呼叫 `ReviewHtmlWriter.WriteAsync("checkout-review.html", "Checkout PR Review", findings)`。
3. 用瀏覽器打開 `checkout-review.html`。
4. 回到 PR 抽查 HTML 提到的檔案與 diff。
5. 確認 HTML 是輔助閱讀，不是唯一事實來源。

注意事項：

- HTML 中呈現的 code snippet 需要經過 HTML encoding，避免特殊字元破壞頁面。
- 不要把 token、密碼、客戶資料或內部敏感資訊放進可公開分享的 HTML。
- 若 HTML 會上傳到 S3 或 GitHub Pages，要先確認 repo 與 bucket 權限。
- 如果 HTML 是一次性 artifact，可以不必追求可維護架構；重點是服務當下任務。

## 範例對應概念說明

上面的例子對應文章中的幾個重點：

- 資訊密度：用表格、顏色、區塊和資料流圖降低閱讀成本。
- 視覺清晰：reviewer 不必在長篇 Markdown 裡找重點。
- 易於分享：HTML 可以作為 PR 附件、內部連結或本機檔案傳給同事。
- 雙向互動：copy button 可以把 HTML 裡整理好的結論轉回 PR comment 或 prompt。
- 上下文 ingestion：Claude Code 能讀取實際 codebase 與 diff，輸出才會貼近專案。

## 如何操作 / 使用端流程

你可以用這個簡化流程判斷要不要請 agent 產 HTML：

```text
任務只有文字結論？ -> 用 Markdown
需要比較、視覺化、互動、分享或跨檔案理解？ -> 考慮 HTML
需要長期維護與精準 diff？ -> Markdown / ADR / 正式文件優先
需要一次性操作介面？ -> HTML 很適合
```

常用 prompt 起手式：

```text
請不要只輸出 Markdown。
請產生一個單檔 HTML artifact，讓我可以用瀏覽器閱讀與操作。
這份 HTML 的讀者是 [角色]。
讀者看完後要能 [決策或行動]。
請包含 [圖表 / diff annotation / 表格 / slider / copy button / flowchart]。
```

## 端到端流程

```text
工作需求 / PR / 研究問題
-> Claude Code 讀取 codebase、diff、issue、log 或文件
-> 產生 HTML artifact
-> 人用瀏覽器閱讀、比較或操作
-> 匯出 summary、prompt、JSON、diff 或 review comment
-> 回到 Claude Code / PR / issue 繼續工作
```

這個流程的重點是讓人保持在 loop 裡。不是讓 agent 自己決定一切，而是讓 agent 把複雜資訊整理成你真的願意讀、能判斷、可回饋的介面。

## 如果結果和預期不同

如果產出的 HTML 很漂亮但不好用，先檢查：

- 你有沒有說清楚讀者是誰。
- 你有沒有說清楚讀者要做什麼決策。
- 你有沒有指定資料來源，例如 diff、檔案、issue 或 log。
- 你有沒有要求它抽查或標註引用來源。
- 它是否只是把 Markdown 包成 HTML，而不是增加視覺結構或互動。
- 它是否過度設計，導致重點反而被動畫、卡片或顏色淹沒。

修正 prompt 範例：

```text
這份 HTML 現在太像漂亮版摘要，請重做。
請把重點放在 reviewer 的決策：
- 這個 PR 主要改了什麼？
- 哪些檔案是高風險？
- 哪些測試一定要補？
- 哪些結論來自哪個 diff 片段？
請減少裝飾，增加資料流圖、風險表與可複製 review comment。
```

## 做完後檢查

完成一份 HTML artifact 後，至少檢查：

- 是否能用瀏覽器直接開啟。
- 是否有明確標題、讀者、任務目標。
- 是否真的比 Markdown 更容易讀，而不是更花。
- 是否有引用實際檔案、diff、資料或來源。
- 是否能支援下一步行動，例如 review、決策、複製 prompt、產出 JSON。
- 是否沒有外洩敏感資訊。
- 若要進 git，是否能接受 HTML diff 較吵的代價。

## 負面例子 / 錯誤用法

錯誤做法：

```text
請把這份 README 變成很炫的 HTML，加很多動畫和漸層。
```

問題：

- 沒有說讀者是誰。
- 沒有說這份 HTML 要幫助做什麼決策。
- 沒有指定資料來源與驗證方式。
- 很容易產生漂亮但空泛的頁面。
- 可能讓閱讀成本變高，反而比 Markdown 更糟。

修正方向：

```text
請把這份 README 轉成 onboarding HTML。
讀者是第一天加入專案的 junior backend engineer。
目標是讓他 15 分鐘內知道如何啟動專案、跑測試、找到主要模組。
請包含：
- 專案地圖
- 本機啟動流程
- 常見錯誤與排查
- 第一個可做的小任務
不要加無關動畫。
```

## 小練習

練習 1：PR 說明 HTML

找一個你最近做過的小 PR，請 Claude Code 產生一份 `pr-explainer.html`。內容要包含：

- 改動摘要。
- 檔案責任地圖。
- 主要資料流。
- reviewer 最需要看的 3 個風險。
- 可複製的 PR description。

練習 2：Prompt 調整工作台

把一段常用 prompt 做成 HTML editor：

- 左邊可編輯 prompt template。
- 右邊放 3 組 sample input。
- 即時顯示填入後結果。
- 加一個 copy final prompt button。

練習 3：Feature flag 編輯器

請 agent 讀一份 JSON feature flag 設定，產生單檔 HTML：

- 依功能區分組。
- 顯示 flag 依賴關係。
- 當 prerequisite 沒開時提示。
- 匯出 changed keys diff。

## Junior 常見誤解

誤解 1：HTML 一定比 Markdown 好。

修正：HTML 適合複雜閱讀、視覺化、互動與分享；短筆記、長期文件、精準 diff 還是 Markdown 很強。

誤解 2：產生 HTML 就是在做前端產品。

修正：這裡的 HTML 很多時候是一次性 artifact，不需要做成可維護產品，也不需要進正式系統。

誤解 3：只要頁面漂亮就成功。

修正：成功標準是讀者能更快理解、比較、判斷與採取下一步。漂亮只是輔助。

誤解 4：HTML 可以取代 review。

修正：HTML 是 review 輔助工具。結論仍要回到原始 diff、測試與需求確認。

誤解 5：生成較慢代表不值得。

修正：如果它能讓多人少花 30 分鐘讀懂一個複雜 PR，花更多生成時間可能很划算。但小任務就不必硬用。

## 一句話總結

當 agent 的輸出不只是「給你一段文字」，而是要幫你理解、比較、互動、分享與回饋時，HTML 可以變成比 Markdown 更有效的工作介面。

## 驗證

- 來源連結：已附上。
- 單篇文章章節：1 / 1。
- 真實工作流程例子：1 / 1。
- 工作任務欄位：1 / 1。
- 你先判斷欄位：1 / 1。
- 會動到欄位：1 / 1。
- 資料怎麼流欄位：1 / 1。
- 流程路線圖欄位：1 / 1。
- 工作中會寫 / 檢查的片段欄位：1 / 1。
- 交付前驗證欄位：1 / 1。
- 常見卡點欄位：1 / 1。
- 範例範圍地圖：1 / 1。
- 完整實作流程：1 / 1。
- 負面例子：1 / 1。
- 小練習：1 / 1。
- Junior 常見誤解：1 / 1。
- 一句話總結：1 / 1。
