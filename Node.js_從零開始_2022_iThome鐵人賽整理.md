# Node.js 從零開始 2022 iThome 鐵人賽整理

來源系列：<https://ithelp.ithome.com.tw/m/users/20119300/ironman/5300>

來源提醒：本筆記整理自 iThome「Node.js 從零開始」系列，共取得 30 / 30 篇。內容以來源主題為主，並加入 2026 補強的工作流程、檔案分層、驗證方式與負面例子，方便 junior developer 實作練習。

## 逐篇處理清單

- Day 1：[Node.js - 從零開始之前](https://ithelp.ithome.com.tw/articles/10291357)
- Day 2：[Node.JS - 簡介](https://ithelp.ithome.com.tw/articles/10292188)
- Day 3：[Node.js - V8 引擎](https://ithelp.ithome.com.tw/articles/10292941)
- Day 4：[Node.js- npm 基礎認識](https://ithelp.ithome.com.tw/articles/10293738)
- Day 5：[Node.js - Hello World！執行 JS 檔案](https://ithelp.ithome.com.tw/articles/10294601)
- Day 6：[Node.js - Global 全域物件](https://ithelp.ithome.com.tw/articles/10295210)
- Day 7：[Node.js - require, module export 模組設計](https://ithelp.ithome.com.tw/articles/10295904)
- Day 8：[Node.js - exports 模組設計](https://ithelp.ithome.com.tw/articles/10296615)
- Day 9：[Node.js - createServer 起手式](https://ithelp.ithome.com.tw/articles/10297261)
- Day 10：[Node.js - createServer 核心知識](https://ithelp.ithome.com.tw/articles/10298176)
- Day 11：[Node.js - __dirname, __filename 取得檔案路徑與檔名](https://ithelp.ithome.com.tw/articles/10298686)
- Day 12：[Node.js - 使用 path 得到更多資訊](https://ithelp.ithome.com.tw/articles/10299383)
- Day 13：[Node.js - 專案必備管理工具 NVM](https://ithelp.ithome.com.tw/articles/10299516)
- Day 14：[Node.js - 引用 Firebase 當作資料庫](https://ithelp.ithome.com.tw/articles/10300731)
- Day 15：[Node.js - ref() 資料路徑、set() 新增資料](https://ithelp.ithome.com.tw/articles/10301289)
- Day 16：[Node.js - once() 顯示資料於網頁上、on() 隨時監聽資料](https://ithelp.ithome.com.tw/articles/10302088)
- Day 17：[Node.js - push() 新增資料](https://ithelp.ithome.com.tw/articles/10302778)
- Day 18：[Node.js - child() 子路徑、remove()刪除資料](https://ithelp.ithome.com.tw/articles/10303313)
- Day 19：[Node.js - 在網頁即時瀏覽 Firebase 的資料](https://ithelp.ithome.com.tw/articles/10303698)
- Day 20：[Node.js - 寫一個簡單的 TodoList：新增資料並渲染於網頁上](https://ithelp.ithome.com.tw/articles/10304257)
- Day 21：[Node.js - 寫一個簡單的 Todolist：刪除資料](https://ithelp.ithome.com.tw/articles/10304832)
- Day 22：[Node.js - 排序功能：orderByChild()排序、forEach()遍歷資料](https://ithelp.ithome.com.tw/articles/10305364)
- Day 23：[Node.js - 搜尋區間 startAt()、endAt()、equalto()](https://ithelp.ithome.com.tw/articles/10305499)
- Day 24：[Node.js - 限制資料筆數 limit()](https://ithelp.ithome.com.tw/articles/10305520)
- Day 25：[Node.js - 使用 reverse() 反轉資料排序](https://ithelp.ithome.com.tw/articles/10306913)
- Day 26：[Node.js - Express.js 介紹與起手式](https://ithelp.ithome.com.tw/articles/10307264)
- Day 27：[Node.js - 路由設計](https://ithelp.ithome.com.tw/articles/10307712)
- Day 28：[Node.js - middleware 基礎介紹](https://ithelp.ithome.com.tw/articles/10308126)
- Day 29：[Node.js - 載入靜態檔案 static](https://ithelp.ithome.com.tw/articles/10308430)
- Day 30：[Node.js - EJS 介紹與起手式](https://ithelp.ithome.com.tw/articles/10308702)

## 主線專案

### 專案最終會長成什麼

主線專案是一個從 Node.js 基礎一路累積到 Express + Firebase + EJS 的 Todo API / 管理頁練習專案。它不是 production-ready 系統，而是用來建立後端入門直覺：如何啟動 Node.js、拆模組、處理 HTTP、接資料庫、渲染畫面，最後能用 Express 提供 API、靜態檔案與 server-side template。

### 需要的檔案地圖

| 區域 | 建議位置 | 負責內容 |
| --- | --- | --- |
| 專案設定 | package.json、.nvmrc、.env.example | Node 版本、npm scripts、環境變數範本 |
| 入口 | src/index.js、src/server.js、src/app.js | CLI、http server 或 Express app 啟動點 |
| 工具模組 | src/utils/* | 日期、文字、路徑等可重用邏輯 |
| 資料層 | src/firebase/database.js、src/todos/* | Firebase 初始化與 Todo CRUD |
| 路由層 | src/routes/* | Express route 與 request/response 對應 |
| 前端靜態檔 | public/index.html、public/app.js | Todo 畫面與瀏覽器端互動 |
| Template | iews/*.ejs | server-side rendered HTML |

### 30 天交付物地圖

- Day 1：學習路線與主線專案規劃
- Day 2：Node.js 是讓 JavaScript 離開瀏覽器、執行在伺服器與本機環境的 runtime
- Day 3：V8 負責解析、編譯與執行 JavaScript，Node.js 則把檔案、網路等能力接到 JS 世界
- Day 4：npm 管理套件、版本、腳本與專案相依性
- Day 5：用 node 指令執行 JavaScript 檔案，確認本機開發流程可運作
- Day 6：global 與 process 提供跨檔案可用的執行環境資訊，但不應濫用共享狀態
- Day 7：CommonJS 用 require 載入模組，用 module.exports 輸出單一公開介面
- Day 8：exports 是 module.exports 的捷徑，適合輸出多個具名功能，但重新指定 exports 會失效
- Day 9：http.createServer 可以建立最小 HTTP server，接住 request 並回傳 response
- Day 10：request 包含方法、URL、headers，response 負責狀態碼、標頭與內容輸出
- Day 11：__dirname 指目前模組所在資料夾，__filename 指目前檔案完整路徑
- Day 12：path 模組用來安全組合、解析與正規化檔案路徑
- Day 13：NVM 用來切換 Node.js 版本，降低不同專案版本衝突
- Day 14：Firebase Realtime Database 可作為練習用即時資料庫，Node.js 透過 SDK 連線
- Day 15：ref 指向資料路徑，set 會覆蓋該路徑資料
- Day 16：once 讀取一次資料，on 持續監聽資料變動
- Day 17：push 會在路徑下產生唯一 key，適合新增多筆同類資料
- Day 18：child 可以組出子節點，remove 刪除指定路徑資料
- Day 19：前端可透過監聽資料變動，把 Firebase 資料即時渲染到畫面
- Day 20：把表單輸入、資料寫入與畫面渲染串成完整 Todo 新增流程
- Day 21：刪除流程要從 UI 事件帶著正確 id 到資料層
- Day 22：orderByChild 依子欄位排序，snapshot.forEach 依排序結果逐筆處理
- Day 23：startAt/endAt/equalTo 用於依排序欄位篩選資料範圍或精準值
- Day 24：limitToFirst/limitToLast 限制回傳筆數，避免一次載入過多資料
- Day 25：Firebase 查詢結果常需轉陣列後 reverse，讓最新資料排前面
- Day 26：Express 封裝 Node http server，讓路由、中介層與回應更容易維護
- Day 27：路由把 URL 與 HTTP method 對應到清楚的處理函式
- Day 28：middleware 是 request 進入 handler 前後的共用處理流程
- Day 29：express.static 讓 server 提供 public 目錄中的 HTML、CSS、JS 等靜態檔案
- Day 30：EJS 是 server-side template engine，可在伺服器產生 HTML

### 主線端到端流程

`	ext
使用者 / CLI / 瀏覽器 -> Node.js 入口 -> 模組或 Express route -> Firebase / 檔案 / 記憶體資料 -> JSON response / HTML / console / Firebase console
`

### 主線做完後檢查

- 
ode -v 與 .nvmrc 對得上。
- 
pm install 後可以執行主要 script。
- /health、/todos、靜態首頁與 EJS 頁面都能回應。
- Firebase 練習資料路徑清楚，不會誤刪正式資料。
- 每個功能都能說明輸入、處理層、輸出與錯誤排查位置。
## Day 1：Node.js - 從零開始之前

來源：[Node.js - 從零開始之前](https://ithelp.ithome.com.tw/articles/10291357)

### 這篇文章主要在講什麼

這一天的主題是「學習路線與主線專案規劃」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能判斷 Node.js 適合拿來做小工具、API 與後端服務，並規劃 30 天學習交付物。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：學習路線與主線專案規劃。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：主管希望前端工程師評估 Node.js 是否能用來做內部資料整理工具與簡單 API，先產出學習路線與驗證計畫。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：README.md、docs/learning-map.md、package.json。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：需求 -> 學習地圖 -> 專案骨架 -> 每日交付物 -> README 檢查。
- 工作中會寫 / 檢查的片段：

`	ext
mkdir node-zero-start`ncd node-zero-start`nnpm init -y`nnode -v`nnpm -v
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「學習路線與主線專案規劃」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | README.md | 接收需求或啟動當天流程 |
| 邏輯 | README.md、docs/learning-map.md、package.json | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
mkdir node-zero-start`ncd node-zero-start`nnpm init -y`nnode -v`nnpm -v
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

學習路線與主線專案規劃，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 2：Node.JS - 簡介

來源：[Node.JS - 簡介](https://ithelp.ithome.com.tw/articles/10292188)

### 這篇文章主要在講什麼

這一天的主題是「Node.js 是讓 JavaScript 離開瀏覽器、執行在伺服器與本機環境的 runtime」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能說明 runtime、非同步 I/O、前端 JS 與 Node.js 的差異。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：Node.js 是讓 JavaScript 離開瀏覽器、執行在伺服器與本機環境的 runtime。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：團隊要把前端資料轉檔流程從手動改成 CLI，小組先確認 Node.js 是否適合執行檔案讀寫與 HTTP 任務。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：scripts/check-runtime.js、README.md。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：需求 -> runtime 判斷 -> node 執行腳本 -> console/log 輸出。
- 工作中會寫 / 檢查的片段：

`	ext
// scripts/check-runtime.js`nconsole.log(``Node runtime: ${process.version}``);`nconsole.log(``Platform: ${process.platform}``);
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「Node.js 是讓 JavaScript 離開瀏覽器、執行在伺服器與本機環境的 runtime」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | scripts/check-runtime.js | 接收需求或啟動當天流程 |
| 邏輯 | scripts/check-runtime.js、README.md | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
// scripts/check-runtime.js`nconsole.log(``Node runtime: ${process.version}``);`nconsole.log(``Platform: ${process.platform}``);
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

Node.js 是讓 JavaScript 離開瀏覽器、執行在伺服器與本機環境的 runtime，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 3：Node.js - V8 引擎

來源：[Node.js - V8 引擎](https://ithelp.ithome.com.tw/articles/10292941)

### 這篇文章主要在講什麼

這一天的主題是「V8 負責解析、編譯與執行 JavaScript，Node.js 則把檔案、網路等能力接到 JS 世界」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能分辨 JS 語言、V8 引擎與 Node.js API 的責任邊界。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：V8 負責解析、編譯與執行 JavaScript，Node.js 則把檔案、網路等能力接到 JS 世界。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：code review 發現有人把瀏覽器 API、V8、Node.js API 混在一起討論，你要補一份責任分界說明。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：docs/runtime-boundary.md、scripts/runtime-check.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：討論問題 -> 責任分界 -> 執行環境檢查 -> 文件交付。
- 工作中會寫 / 檢查的片段：

`	ext
console.log(typeof fetch);`nconsole.log(typeof require);`nconsole.log(typeof document); // Node.js 預設沒有瀏覽器 DOM
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「V8 負責解析、編譯與執行 JavaScript，Node.js 則把檔案、網路等能力接到 JS 世界」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | docs/runtime-boundary.md | 接收需求或啟動當天流程 |
| 邏輯 | docs/runtime-boundary.md、scripts/runtime-check.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
console.log(typeof fetch);`nconsole.log(typeof require);`nconsole.log(typeof document); // Node.js 預設沒有瀏覽器 DOM
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

V8 負責解析、編譯與執行 JavaScript，Node.js 則把檔案、網路等能力接到 JS 世界，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 4：Node.js- npm 基礎認識

來源：[Node.js- npm 基礎認識](https://ithelp.ithome.com.tw/articles/10293738)

### 這篇文章主要在講什麼

這一天的主題是「npm 管理套件、版本、腳本與專案相依性」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能建立 package.json、安裝套件、分辨 dependencies 與 devDependencies。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：npm 管理套件、版本、腳本與專案相依性。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：專案需要加入日期格式化套件與開發用 nodemon，並讓新人可以用 npm scripts 啟動。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：package.json、package-lock.json、src/index.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：需求 -> npm install -> package.json scripts -> node 執行 -> lockfile 檢查。
- 工作中會寫 / 檢查的片段：

`	ext
npm init -y`nnpm install dayjs`nnpm install -D nodemon`nnpm pkg set scripts.dev="nodemon src/index.js"
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「npm 管理套件、版本、腳本與專案相依性」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | package.json | 接收需求或啟動當天流程 |
| 邏輯 | package.json、package-lock.json、src/index.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
npm init -y`nnpm install dayjs`nnpm install -D nodemon`nnpm pkg set scripts.dev="nodemon src/index.js"
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

npm 管理套件、版本、腳本與專案相依性，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 5：Node.js - Hello World！執行 JS 檔案

來源：[Node.js - Hello World！執行 JS 檔案](https://ithelp.ithome.com.tw/articles/10294601)

### 這篇文章主要在講什麼

這一天的主題是「用 node 指令執行 JavaScript 檔案，確認本機開發流程可運作」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能建立 JS 檔、從終端機執行、理解入口檔概念。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：用 node 指令執行 JavaScript 檔案，確認本機開發流程可運作。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：你要建立第一個可執行入口，讓團隊確認環境裝好，並輸出目前工具版本。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/index.js、package.json。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：需求 -> 建立入口檔 -> node src/index.js -> npm script -> 輸出結果。
- 工作中會寫 / 檢查的片段：

`	ext
// src/index.js`nconsole.log("Hello Node.js");`nconsole.log(``目前版本：${process.version}``);
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「用 node 指令執行 JavaScript 檔案，確認本機開發流程可運作」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/index.js | 接收需求或啟動當天流程 |
| 邏輯 | src/index.js、package.json | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
// src/index.js`nconsole.log("Hello Node.js");`nconsole.log(``目前版本：${process.version}``);
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

用 node 指令執行 JavaScript 檔案，確認本機開發流程可運作，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 6：Node.js - Global 全域物件

來源：[Node.js - Global 全域物件](https://ithelp.ithome.com.tw/articles/10295210)

### 這篇文章主要在講什麼

這一天的主題是「global 與 process 提供跨檔案可用的執行環境資訊，但不應濫用共享狀態」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能使用 process.env、process.argv，並知道不要把業務資料塞進 global。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：global 與 process 提供跨檔案可用的執行環境資訊，但不應濫用共享狀態。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：內部工具要從命令列讀取環境名稱，依 dev 或 prod 印出不同提示。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/env.js、package.json、.env.example。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：CLI 參數 -> process.argv/process.env -> 設定判斷 -> console 輸出。
- 工作中會寫 / 檢查的片段：

`	ext
const env = process.argv[2] ?? process.env.NODE_ENV ?? "dev";`nconsole.log(``目前環境：${env}``);
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「global 與 process 提供跨檔案可用的執行環境資訊，但不應濫用共享狀態」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/env.js | 接收需求或啟動當天流程 |
| 邏輯 | src/env.js、package.json、.env.example | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
const env = process.argv[2] ?? process.env.NODE_ENV ?? "dev";`nconsole.log(``目前環境：${env}``);
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

global 與 process 提供跨檔案可用的執行環境資訊，但不應濫用共享狀態，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 7：Node.js - require, module export 模組設計

來源：[Node.js - require, module export 模組設計](https://ithelp.ithome.com.tw/articles/10295904)

### 這篇文章主要在講什麼

這一天的主題是「CommonJS 用 require 載入模組，用 module.exports 輸出單一公開介面」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能把功能拆成模組，並由入口檔呼叫。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：CommonJS 用 require 載入模組，用 module.exports 輸出單一公開介面。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：Todo 工具的格式化邏輯太散，請把日期格式化抽到 utils，再由主程式呼叫。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/index.js、src/utils/dateFormatter.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：需求 -> 拆模組 -> module.exports -> require -> 執行驗證。
- 工作中會寫 / 檢查的片段：

`	ext
// src/utils/dateFormatter.js`nmodule.exports = function formatDate(date) {`n  return date.toISOString().slice(0, 10);`n};`n// src/index.js`nconst formatDate = require("./utils/dateFormatter");`nconsole.log(formatDate(new Date()));
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「CommonJS 用 require 載入模組，用 module.exports 輸出單一公開介面」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/index.js | 接收需求或啟動當天流程 |
| 邏輯 | src/index.js、src/utils/dateFormatter.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
// src/utils/dateFormatter.js`nmodule.exports = function formatDate(date) {`n  return date.toISOString().slice(0, 10);`n};`n// src/index.js`nconst formatDate = require("./utils/dateFormatter");`nconsole.log(formatDate(new Date()));
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

CommonJS 用 require 載入模組，用 module.exports 輸出單一公開介面，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 8：Node.js - exports 模組設計

來源：[Node.js - exports 模組設計](https://ithelp.ithome.com.tw/articles/10296615)

### 這篇文章主要在講什麼

這一天的主題是「exports 是 module.exports 的捷徑，適合輸出多個具名功能，但重新指定 exports 會失效」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能分辨 exports.foo 與 module.exports 的差異。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：exports 是 module.exports 的捷徑，適合輸出多個具名功能，但重新指定 exports 會失效。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：Todo 工具需要同時輸出 validateTitle 與 normalizeTitle，請設計具名工具模組。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/utils/todoText.js、src/index.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：需求 -> exports 多函式 -> require 物件 -> 呼叫具名方法。
- 工作中會寫 / 檢查的片段：

`	ext
exports.normalizeTitle = title => title.trim();`nexports.validateTitle = title => title.trim().length > 0;
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「exports 是 module.exports 的捷徑，適合輸出多個具名功能，但重新指定 exports 會失效」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/utils/todoText.js | 接收需求或啟動當天流程 |
| 邏輯 | src/utils/todoText.js、src/index.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
exports.normalizeTitle = title => title.trim();`nexports.validateTitle = title => title.trim().length > 0;
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

exports 是 module.exports 的捷徑，適合輸出多個具名功能，但重新指定 exports 會失效，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 9：Node.js - createServer 起手式

來源：[Node.js - createServer 起手式](https://ithelp.ithome.com.tw/articles/10297261)

### 這篇文章主要在講什麼

這一天的主題是「http.createServer 可以建立最小 HTTP server，接住 request 並回傳 response」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能建立本機 server、設定 status code 與 response body。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：http.createServer 可以建立最小 HTTP server，接住 request 並回傳 response。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：PM 想先看 Todo API 是否能通，請做一個 /health 檢查端點。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/server.js、package.json。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：瀏覽器/request -> http server -> route 判斷 -> response -> curl 驗證。
- 工作中會寫 / 檢查的片段：

`	ext
const http = require("http");`nconst server = http.createServer((req, res) => {`n  res.writeHead(200, { "Content-Type": "application/json" });`n  res.end(JSON.stringify({ status: "ok" }));`n});`nserver.listen(3000);
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「http.createServer 可以建立最小 HTTP server，接住 request 並回傳 response」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/server.js | 接收需求或啟動當天流程 |
| 邏輯 | src/server.js、package.json | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
const http = require("http");`nconst server = http.createServer((req, res) => {`n  res.writeHead(200, { "Content-Type": "application/json" });`n  res.end(JSON.stringify({ status: "ok" }));`n});`nserver.listen(3000);
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

http.createServer 可以建立最小 HTTP server，接住 request 並回傳 response，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 10：Node.js - createServer 核心知識

來源：[Node.js - createServer 核心知識](https://ithelp.ithome.com.tw/articles/10298176)

### 這篇文章主要在講什麼

這一天的主題是「request 包含方法、URL、headers，response 負責狀態碼、標頭與內容輸出」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能依 URL 與 method 回傳不同結果，並處理 404。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：request 包含方法、URL、headers，response 負責狀態碼、標頭與內容輸出。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：QA 回報任何路徑都回 ok，請補上 method/path 判斷與 404。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/server.js、tests/http-check.http。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：HTTP request -> req.method/req.url -> route handler -> response status -> 測試。
- 工作中會寫 / 檢查的片段：

`	ext
if (req.method === "GET" && req.url === "/health") {`n  res.writeHead(200, { "Content-Type": "application/json" });`n  return res.end(JSON.stringify({ status: "ok" }));`n}`nres.writeHead(404);`nres.end("Not Found");
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「request 包含方法、URL、headers，response 負責狀態碼、標頭與內容輸出」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/server.js | 接收需求或啟動當天流程 |
| 邏輯 | src/server.js、tests/http-check.http | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
if (req.method === "GET" && req.url === "/health") {`n  res.writeHead(200, { "Content-Type": "application/json" });`n  return res.end(JSON.stringify({ status: "ok" }));`n}`nres.writeHead(404);`nres.end("Not Found");
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

request 包含方法、URL、headers，response 負責狀態碼、標頭與內容輸出，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 11：Node.js - __dirname, __filename 取得檔案路徑與檔名

來源：[Node.js - __dirname, __filename 取得檔案路徑與檔名](https://ithelp.ithome.com.tw/articles/10298686)

### 這篇文章主要在講什麼

這一天的主題是「__dirname 指目前模組所在資料夾，__filename 指目前檔案完整路徑」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能避免因執行工作目錄不同導致讀不到檔案。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：__dirname 指目前模組所在資料夾，__filename 指目前檔案完整路徑。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：部署後讀不到 config.json，請改成從檔案所在目錄組路徑。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/config/loadConfig.js、config/app.json。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：啟動 server -> __dirname 定位 -> fs 讀設定 -> service 使用設定。
- 工作中會寫 / 檢查的片段：

`	ext
const path = require("path");`nconst configPath = path.join(__dirname, "../../config/app.json");`nconsole.log(configPath);
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「__dirname 指目前模組所在資料夾，__filename 指目前檔案完整路徑」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/config/loadConfig.js | 接收需求或啟動當天流程 |
| 邏輯 | src/config/loadConfig.js、config/app.json | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
const path = require("path");`nconst configPath = path.join(__dirname, "../../config/app.json");`nconsole.log(configPath);
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

__dirname 指目前模組所在資料夾，__filename 指目前檔案完整路徑，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 12：Node.js - 使用 path 得到更多資訊

來源：[Node.js - 使用 path 得到更多資訊](https://ithelp.ithome.com.tw/articles/10299383)

### 這篇文章主要在講什麼

這一天的主題是「path 模組用來安全組合、解析與正規化檔案路徑」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能使用 path.join、path.resolve、path.extname、path.basename。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：path 模組用來安全組合、解析與正規化檔案路徑。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：上傳檔案清單要顯示檔名與副檔名，且不能用字串切割路徑。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/files/fileInfo.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：檔案路徑輸入 -> path parse -> 格式化資料 -> API/console 輸出。
- 工作中會寫 / 檢查的片段：

`	ext
const path = require("path");`nconst file = "uploads/report.csv";`nconsole.log(path.basename(file));`nconsole.log(path.extname(file));
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「path 模組用來安全組合、解析與正規化檔案路徑」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/files/fileInfo.js | 接收需求或啟動當天流程 |
| 邏輯 | src/files/fileInfo.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
const path = require("path");`nconst file = "uploads/report.csv";`nconsole.log(path.basename(file));`nconsole.log(path.extname(file));
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

path 模組用來安全組合、解析與正規化檔案路徑，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 13：Node.js - 專案必備管理工具 NVM

來源：[Node.js - 專案必備管理工具 NVM](https://ithelp.ithome.com.tw/articles/10299516)

### 這篇文章主要在講什麼

這一天的主題是「NVM 用來切換 Node.js 版本，降低不同專案版本衝突」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能用 .nvmrc 固定專案版本並提醒團隊切換。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：NVM 用來切換 Node.js 版本，降低不同專案版本衝突。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：同事 Node 版本不同導致套件安裝失敗，請加上版本規範與啟動檢查。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：.nvmrc、README.md、package.json。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：clone 專案 -> nvm use -> npm install -> npm run dev -> 版本一致。
- 工作中會寫 / 檢查的片段：

`	ext
node -v`nnvm install 20`nnvm use 20`nnode -v
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「NVM 用來切換 Node.js 版本，降低不同專案版本衝突」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | .nvmrc | 接收需求或啟動當天流程 |
| 邏輯 | .nvmrc、README.md、package.json | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
node -v`nnvm install 20`nnvm use 20`nnode -v
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

NVM 用來切換 Node.js 版本，降低不同專案版本衝突，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 14：Node.js - 引用 Firebase 當作資料庫

來源：[Node.js - 引用 Firebase 當作資料庫](https://ithelp.ithome.com.tw/articles/10300731)

### 這篇文章主要在講什麼

這一天的主題是「Firebase Realtime Database 可作為練習用即時資料庫，Node.js 透過 SDK 連線」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能初始化 Firebase 設定，並把資料庫連線封裝起來。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：Firebase Realtime Database 可作為練習用即時資料庫，Node.js 透過 SDK 連線。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：Todo 小專案要從記憶體改成 Firebase，先建立可替換的 database 模組。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/firebase/database.js、.env.example。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：環境設定 -> Firebase init -> database reference -> service 呼叫。
- 工作中會寫 / 檢查的片段：

`	ext
const firebase = require("firebase/app");`nrequire("firebase/database");`nfirebase.initializeApp({ databaseURL: process.env.FIREBASE_DB_URL });`nmodule.exports = firebase.database();
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「Firebase Realtime Database 可作為練習用即時資料庫，Node.js 透過 SDK 連線」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/firebase/database.js | 接收需求或啟動當天流程 |
| 邏輯 | src/firebase/database.js、.env.example | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
const firebase = require("firebase/app");`nrequire("firebase/database");`nfirebase.initializeApp({ databaseURL: process.env.FIREBASE_DB_URL });`nmodule.exports = firebase.database();
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

Firebase Realtime Database 可作為練習用即時資料庫，Node.js 透過 SDK 連線，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 15：Node.js - ref() 資料路徑、set() 新增資料

來源：[Node.js - ref() 資料路徑、set() 新增資料](https://ithelp.ithome.com.tw/articles/10301289)

### 這篇文章主要在講什麼

這一天的主題是「ref 指向資料路徑，set 會覆蓋該路徑資料」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能選擇資料路徑並寫入 Todo 資料。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：ref 指向資料路徑，set 會覆蓋該路徑資料。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：新增一筆預設 Todo 到 /todos/demo，讓資料庫連線先被驗證。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/todos/createDemoTodo.js、src/firebase/database.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：service 呼叫 -> db.ref("todos/demo") -> set(data) -> Firebase console 檢查。
- 工作中會寫 / 檢查的片段：

`	ext
await db.ref("todos/demo").set({`n  title: "learn Node.js",`n  done: false`n});
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「ref 指向資料路徑，set 會覆蓋該路徑資料」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/todos/createDemoTodo.js | 接收需求或啟動當天流程 |
| 邏輯 | src/todos/createDemoTodo.js、src/firebase/database.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
await db.ref("todos/demo").set({`n  title: "learn Node.js",`n  done: false`n});
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

ref 指向資料路徑，set 會覆蓋該路徑資料，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 16：Node.js - once() 顯示資料於網頁上、on() 隨時監聽資料

來源：[Node.js - once() 顯示資料於網頁上、on() 隨時監聽資料](https://ithelp.ithome.com.tw/articles/10302088)

### 這篇文章主要在講什麼

這一天的主題是「once 讀取一次資料，on 持續監聽資料變動」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能依需求選擇一次性查詢或即時監聽。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：once 讀取一次資料，on 持續監聽資料變動。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：管理頁第一次進入要讀取 Todo，開發偵錯頁則要即時看到資料變動。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/todos/readTodos.js、public/index.html。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：頁面載入 -> once/on -> snapshot -> render -> Firebase 更新。
- 工作中會寫 / 檢查的片段：

`	ext
db.ref("todos").once("value", snapshot => {`n  console.log(snapshot.val());`n});`ndb.ref("todos").on("value", snapshot => {`n  render(snapshot.val());`n});
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「once 讀取一次資料，on 持續監聽資料變動」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/todos/readTodos.js | 接收需求或啟動當天流程 |
| 邏輯 | src/todos/readTodos.js、public/index.html | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
db.ref("todos").once("value", snapshot => {`n  console.log(snapshot.val());`n});`ndb.ref("todos").on("value", snapshot => {`n  render(snapshot.val());`n});
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

once 讀取一次資料，on 持續監聽資料變動，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 17：Node.js - push() 新增資料

來源：[Node.js - push() 新增資料](https://ithelp.ithome.com.tw/articles/10302778)

### 這篇文章主要在講什麼

這一天的主題是「push 會在路徑下產生唯一 key，適合新增多筆同類資料」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能新增 Todo 並保存 Firebase 產生的 key。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：push 會在路徑下產生唯一 key，適合新增多筆同類資料。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：使用者新增 Todo 時不能覆蓋舊資料，請改用 push 新增。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/todos/addTodo.js、public/app.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：表單 submit -> addTodo -> ref("todos").push -> key 回傳 -> UI 更新。
- 工作中會寫 / 檢查的片段：

`	ext
const result = await db.ref("todos").push({`n  title: inputTitle,`n  done: false,`n  createdAt: Date.now()`n});`nconsole.log(result.key);
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「push 會在路徑下產生唯一 key，適合新增多筆同類資料」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/todos/addTodo.js | 接收需求或啟動當天流程 |
| 邏輯 | src/todos/addTodo.js、public/app.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
const result = await db.ref("todos").push({`n  title: inputTitle,`n  done: false,`n  createdAt: Date.now()`n});`nconsole.log(result.key);
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

push 會在路徑下產生唯一 key，適合新增多筆同類資料，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 18：Node.js - child() 子路徑、remove()刪除資料

來源：[Node.js - child() 子路徑、remove()刪除資料](https://ithelp.ithome.com.tw/articles/10303313)

### 這篇文章主要在講什麼

這一天的主題是「child 可以組出子節點，remove 刪除指定路徑資料」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能依 Todo id 刪除資料而不影響其他節點。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：child 可以組出子節點，remove 刪除指定路徑資料。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：QA 回報刪除按鈕會清掉整份 todos，請改成只刪除指定 id。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/todos/deleteTodo.js、public/app.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：刪除按鈕 -> todoId -> ref("todos").child(id).remove -> DB 檢查。
- 工作中會寫 / 檢查的片段：

`	ext
async function deleteTodo(todoId) {`n  await db.ref("todos").child(todoId).remove();`n}
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「child 可以組出子節點，remove 刪除指定路徑資料」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/todos/deleteTodo.js | 接收需求或啟動當天流程 |
| 邏輯 | src/todos/deleteTodo.js、public/app.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
async function deleteTodo(todoId) {`n  await db.ref("todos").child(todoId).remove();`n}
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

child 可以組出子節點，remove 刪除指定路徑資料，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 19：Node.js - 在網頁即時瀏覽 Firebase 的資料

來源：[Node.js - 在網頁即時瀏覽 Firebase 的資料](https://ithelp.ithome.com.tw/articles/10303698)

### 這篇文章主要在講什麼

這一天的主題是「前端可透過監聽資料變動，把 Firebase 資料即時渲染到畫面」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能把資料快照轉成畫面清單。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：前端可透過監聽資料變動，把 Firebase 資料即時渲染到畫面。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：客服想開一個即時 Todo 看板，不重整也能看到資料新增刪除。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：public/index.html、public/app.js、src/server.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：Firebase on(value) -> snapshot -> Object.entries -> DOM render -> 使用者看板。
- 工作中會寫 / 檢查的片段：

`	ext
function renderTodos(data = {}) {`n  list.innerHTML = Object.entries(data)`n    .map(([id, todo]) => `<li data-id="${id}">${todo.title}</li>`)`n    .join("");`n}
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「前端可透過監聽資料變動，把 Firebase 資料即時渲染到畫面」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | public/index.html | 接收需求或啟動當天流程 |
| 邏輯 | public/index.html、public/app.js、src/server.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
function renderTodos(data = {}) {`n  list.innerHTML = Object.entries(data)`n    .map(([id, todo]) => `<li data-id="${id}">${todo.title}</li>`)`n    .join("");`n}
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

前端可透過監聽資料變動，把 Firebase 資料即時渲染到畫面，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 20：Node.js - 寫一個簡單的 TodoList：新增資料並渲染於網頁上

來源：[Node.js - 寫一個簡單的 TodoList：新增資料並渲染於網頁上](https://ithelp.ithome.com.tw/articles/10304257)

### 這篇文章主要在講什麼

這一天的主題是「把表單輸入、資料寫入與畫面渲染串成完整 Todo 新增流程」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能完成新增 Todo 的端到端流程。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：把表單輸入、資料寫入與畫面渲染串成完整 Todo 新增流程。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：PM 要求第一版 TodoList 能新增任務並立即顯示在頁面。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：public/index.html、public/app.js、src/todos/addTodo.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：使用者輸入 -> submit handler -> Firebase push -> on(value) -> render。
- 工作中會寫 / 檢查的片段：

`	ext
form.addEventListener("submit", async event => {`n  event.preventDefault();`n  await addTodo(input.value.trim());`n  input.value = "";`n});
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「把表單輸入、資料寫入與畫面渲染串成完整 Todo 新增流程」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | public/index.html | 接收需求或啟動當天流程 |
| 邏輯 | public/index.html、public/app.js、src/todos/addTodo.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
form.addEventListener("submit", async event => {`n  event.preventDefault();`n  await addTodo(input.value.trim());`n  input.value = "";`n});
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

把表單輸入、資料寫入與畫面渲染串成完整 Todo 新增流程，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 21：Node.js - 寫一個簡單的 Todolist：刪除資料

來源：[Node.js - 寫一個簡單的 Todolist：刪除資料](https://ithelp.ithome.com.tw/articles/10304832)

### 這篇文章主要在講什麼

這一天的主題是「刪除流程要從 UI 事件帶著正確 id 到資料層」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能在畫面點刪除並同步移除 Firebase 資料。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：刪除流程要從 UI 事件帶著正確 id 到資料層。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：使用者可以刪除已完成或輸入錯誤的 Todo，刪除後列表要更新。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：public/app.js、src/todos/deleteTodo.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：click delete -> dataset.id -> remove -> Firebase event -> rerender。
- 工作中會寫 / 檢查的片段：

`	ext
list.addEventListener("click", event => {`n  const id = event.target.dataset.deleteId;`n  if (id) deleteTodo(id);`n});
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「刪除流程要從 UI 事件帶著正確 id 到資料層」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | public/app.js | 接收需求或啟動當天流程 |
| 邏輯 | public/app.js、src/todos/deleteTodo.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
list.addEventListener("click", event => {`n  const id = event.target.dataset.deleteId;`n  if (id) deleteTodo(id);`n});
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

刪除流程要從 UI 事件帶著正確 id 到資料層，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 22：Node.js - 排序功能：orderByChild()排序、forEach()遍歷資料

來源：[Node.js - 排序功能：orderByChild()排序、forEach()遍歷資料](https://ithelp.ithome.com.tw/articles/10305364)

### 這篇文章主要在講什麼

這一天的主題是「orderByChild 依子欄位排序，snapshot.forEach 依排序結果逐筆處理」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能依 createdAt 或 priority 排序 Todo。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：orderByChild 依子欄位排序，snapshot.forEach 依排序結果逐筆處理。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：使用者希望最新建立的 Todo 可以依建立時間排序顯示。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/todos/listTodos.js、public/app.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：查詢需求 -> orderByChild("createdAt") -> snapshot.forEach -> array -> render。
- 工作中會寫 / 檢查的片段：

`	ext
db.ref("todos")`n  .orderByChild("createdAt")`n  .once("value", snapshot => {`n    const todos = [];`n    snapshot.forEach(child => todos.push({ id: child.key, ...child.val() }));`n    console.log(todos);`n  });
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「orderByChild 依子欄位排序，snapshot.forEach 依排序結果逐筆處理」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/todos/listTodos.js | 接收需求或啟動當天流程 |
| 邏輯 | src/todos/listTodos.js、public/app.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
db.ref("todos")`n  .orderByChild("createdAt")`n  .once("value", snapshot => {`n    const todos = [];`n    snapshot.forEach(child => todos.push({ id: child.key, ...child.val() }));`n    console.log(todos);`n  });
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

orderByChild 依子欄位排序，snapshot.forEach 依排序結果逐筆處理，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 23：Node.js - 搜尋區間 startAt()、endAt()、equalto()

來源：[Node.js - 搜尋區間 startAt()、endAt()、equalto()](https://ithelp.ithome.com.tw/articles/10305499)

### 這篇文章主要在講什麼

這一天的主題是「startAt/endAt/equalTo 用於依排序欄位篩選資料範圍或精準值」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能依狀態、優先級或關鍵字做簡單查詢。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：startAt/endAt/equalTo 用於依排序欄位篩選資料範圍或精準值。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：營運想只看未完成 Todo，請用 done 欄位做 equalTo 查詢。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/todos/searchTodos.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：查詢條件 -> orderByChild -> equalTo/startAt/endAt -> snapshot -> 結果清單。
- 工作中會寫 / 檢查的片段：

`	ext
db.ref("todos")`n  .orderByChild("done")`n  .equalTo(false)`n  .once("value", snapshot => console.log(snapshot.val()));
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「startAt/endAt/equalTo 用於依排序欄位篩選資料範圍或精準值」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/todos/searchTodos.js | 接收需求或啟動當天流程 |
| 邏輯 | src/todos/searchTodos.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
db.ref("todos")`n  .orderByChild("done")`n  .equalTo(false)`n  .once("value", snapshot => console.log(snapshot.val()));
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

startAt/endAt/equalTo 用於依排序欄位篩選資料範圍或精準值，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 24：Node.js - 限制資料筆數 limit()

來源：[Node.js - 限制資料筆數 limit()](https://ithelp.ithome.com.tw/articles/10305520)

### 這篇文章主要在講什麼

這一天的主題是「limitToFirst/limitToLast 限制回傳筆數，避免一次載入過多資料」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能做出最新 N 筆或前 N 筆列表。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：limitToFirst/limitToLast 限制回傳筆數，避免一次載入過多資料。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：首頁只顯示最新 10 筆 Todo，完整列表留給管理頁。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/todos/listRecentTodos.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：首頁載入 -> orderByChild -> limitToLast(10) -> render -> 分頁策略。
- 工作中會寫 / 檢查的片段：

`	ext
db.ref("todos")`n  .orderByChild("createdAt")`n  .limitToLast(10)`n  .once("value", snapshot => console.log(snapshot.val()));
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「limitToFirst/limitToLast 限制回傳筆數，避免一次載入過多資料」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/todos/listRecentTodos.js | 接收需求或啟動當天流程 |
| 邏輯 | src/todos/listRecentTodos.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
db.ref("todos")`n  .orderByChild("createdAt")`n  .limitToLast(10)`n  .once("value", snapshot => console.log(snapshot.val()));
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

limitToFirst/limitToLast 限制回傳筆數，避免一次載入過多資料，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 25：Node.js - 使用 reverse() 反轉資料排序

來源：[Node.js - 使用 reverse() 反轉資料排序](https://ithelp.ithome.com.tw/articles/10306913)

### 這篇文章主要在講什麼

這一天的主題是「Firebase 查詢結果常需轉陣列後 reverse，讓最新資料排前面」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能理解資料庫排序與畫面排序的責任差異。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：Firebase 查詢結果常需轉陣列後 reverse，讓最新資料排前面。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：首頁最新 Todo 目前排在最下方，請在渲染前反轉順序。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/todos/listRecentTodos.js、public/app.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：Firebase 排序結果 -> array -> reverse -> render -> UI 驗證。
- 工作中會寫 / 檢查的片段：

`	ext
const items = [];`nsnapshot.forEach(child => items.push({ id: child.key, ...child.val() }));`nconst newestFirst = items.reverse();
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「Firebase 查詢結果常需轉陣列後 reverse，讓最新資料排前面」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/todos/listRecentTodos.js | 接收需求或啟動當天流程 |
| 邏輯 | src/todos/listRecentTodos.js、public/app.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
const items = [];`nsnapshot.forEach(child => items.push({ id: child.key, ...child.val() }));`nconst newestFirst = items.reverse();
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

Firebase 查詢結果常需轉陣列後 reverse，讓最新資料排前面，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 26：Node.js - Express.js 介紹與起手式

來源：[Node.js - Express.js 介紹與起手式](https://ithelp.ithome.com.tw/articles/10307264)

### 這篇文章主要在講什麼

這一天的主題是「Express 封裝 Node http server，讓路由、中介層與回應更容易維護」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能建立 Express app、啟動 server、回傳 JSON。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：Express 封裝 Node http server，讓路由、中介層與回應更容易維護。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：手寫 http route 開始變複雜，請改成 Express 做 Todo API 起手式。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/app.js、src/server.js、package.json。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：HTTP request -> Express app -> route -> handler -> JSON response。
- 工作中會寫 / 檢查的片段：

`	ext
const express = require("express");`nconst app = express();`napp.get("/health", (req, res) => res.json({ status: "ok" }));`napp.listen(3000);
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「Express 封裝 Node http server，讓路由、中介層與回應更容易維護」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/app.js | 接收需求或啟動當天流程 |
| 邏輯 | src/app.js、src/server.js、package.json | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
const express = require("express");`nconst app = express();`napp.get("/health", (req, res) => res.json({ status: "ok" }));`napp.listen(3000);
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

Express 封裝 Node http server，讓路由、中介層與回應更容易維護，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 27：Node.js - 路由設計

來源：[Node.js - 路由設計](https://ithelp.ithome.com.tw/articles/10307712)

### 這篇文章主要在講什麼

這一天的主題是「路由把 URL 與 HTTP method 對應到清楚的處理函式」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能設計 GET/POST/DELETE /todos 路由。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：路由把 URL 與 HTTP method 對應到清楚的處理函式。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：Todo API 要給前端串接，請規劃 REST-like 路由。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/routes/todos.js、src/app.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：request method/path -> router -> controller/handler -> service -> response。
- 工作中會寫 / 檢查的片段：

`	ext
const router = require("express").Router();`nrouter.get("/", listTodos);`nrouter.post("/", createTodo);`nrouter.delete("/:id", deleteTodo);`nmodule.exports = router;
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「路由把 URL 與 HTTP method 對應到清楚的處理函式」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/routes/todos.js | 接收需求或啟動當天流程 |
| 邏輯 | src/routes/todos.js、src/app.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
const router = require("express").Router();`nrouter.get("/", listTodos);`nrouter.post("/", createTodo);`nrouter.delete("/:id", deleteTodo);`nmodule.exports = router;
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

路由把 URL 與 HTTP method 對應到清楚的處理函式，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 28：Node.js - middleware 基礎介紹

來源：[Node.js - middleware 基礎介紹](https://ithelp.ithome.com.tw/articles/10308126)

### 這篇文章主要在講什麼

這一天的主題是「middleware 是 request 進入 handler 前後的共用處理流程」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能使用 express.json、記錄 log、處理錯誤。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：middleware 是 request 進入 handler 前後的共用處理流程。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：POST /todos 收不到 request body，請加入 JSON middleware 並補 log。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/app.js、src/middleware/requestLogger.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：request -> middleware -> route handler -> error middleware -> response/log。
- 工作中會寫 / 檢查的片段：

`	ext
app.use(express.json());`napp.use((req, res, next) => {`n  console.log(`${req.method} ${req.path}`);`n  next();`n});
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「middleware 是 request 進入 handler 前後的共用處理流程」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/app.js | 接收需求或啟動當天流程 |
| 邏輯 | src/app.js、src/middleware/requestLogger.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
app.use(express.json());`napp.use((req, res, next) => {`n  console.log(`${req.method} ${req.path}`);`n  next();`n});
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

middleware 是 request 進入 handler 前後的共用處理流程，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 29：Node.js - 載入靜態檔案 static

來源：[Node.js - 載入靜態檔案 static](https://ithelp.ithome.com.tw/articles/10308430)

### 這篇文章主要在講什麼

這一天的主題是「express.static 讓 server 提供 public 目錄中的 HTML、CSS、JS 等靜態檔案」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能把前端頁面與 API 放在同一個 Express 專案中練習。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：express.static 讓 server 提供 public 目錄中的 HTML、CSS、JS 等靜態檔案。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：前端 Todo 頁面要由同一個 Node server 提供，不再手動開 HTML 檔。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：public/index.html、public/app.js、src/app.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：browser GET / -> express.static -> public/index.html -> fetch API -> render。
- 工作中會寫 / 檢查的片段：

`	ext
const path = require("path");`napp.use(express.static(path.join(__dirname, "../public")));
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「express.static 讓 server 提供 public 目錄中的 HTML、CSS、JS 等靜態檔案」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | public/index.html | 接收需求或啟動當天流程 |
| 邏輯 | public/index.html、public/app.js、src/app.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
const path = require("path");`napp.use(express.static(path.join(__dirname, "../public")));
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

express.static 讓 server 提供 public 目錄中的 HTML、CSS、JS 等靜態檔案，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## Day 30：Node.js - EJS 介紹與起手式

來源：[Node.js - EJS 介紹與起手式](https://ithelp.ithome.com.tw/articles/10308702)

### 這篇文章主要在講什麼

這一天的主題是「EJS 是 server-side template engine，可在伺服器產生 HTML」。原文從入門角度切入，重點不是背 API 名稱，而是知道它在 Node.js 專案裡負責哪一段工作。

### 為什麼需要這個概念

沒有這個概念時，junior 很容易只會複製範例，卻不知道資料從哪裡進來、哪一層該改、錯誤要先查終端機、瀏覽器還是資料庫。學會它後，才有辦法把 Node.js 從單一片段變成可交付的小功能。

### 學完這篇你應該會做到什麼

- 能設定 view engine、render EJS，理解 SSR 與 API/前端渲染差異。
- 能說明這一天在主線 Todo API / 小工具專案中接到哪一段。
- 能用最小可驗證範例證明功能真的有跑起來。

### 核心重點

- 直覺理解：EJS 是 server-side template engine，可在伺服器產生 HTML。
- 實務價值：它幫你把需求拆到正確的位置，而不是所有程式都塞在單一檔案。
- 風險提醒：範例可用來學習流程，但正式專案還要補環境設定、錯誤處理、測試與安全性。

### 真實工作流程例子

- 工作任務：主管想要一個不依賴前端打包的 Todo 報表頁，請用 EJS 由 server 產生 HTML。
- 你先判斷：先判斷這是環境、模組、資料庫、HTTP route、middleware、靜態檔案或 template 的哪一層責任，再決定要改檔案或設定。
- 會動到：src/app.js、views/todos.ejs、src/routes/pages.js。
- 資料怎麼流：輸入從需求、CLI、HTTP request、表單或 Firebase 資料進來，經過這一天對應的 Node.js 模組或 Express handler，最後在 console、瀏覽器、API response 或 Firebase console 看到結果。
- 流程路線圖：browser request -> Express route -> res.render -> EJS template -> HTML response。
- 工作中會寫 / 檢查的片段：

`	ext
app.set("view engine", "ejs");`napp.get("/todos-page", async (req, res) => {`n  res.render("todos", { todos: await listTodos() });`n});
`

- 交付前驗證：確認 happy path 有輸出預期結果；再測錯誤路徑，例如缺少參數、路由不存在、資料為空、Node 版本不同或 Firebase 權限不足。
- 常見卡點：卡住時先看終端機錯誤訊息，再確認執行目錄、檔案路徑、套件是否安裝、環境變數是否存在，以及瀏覽器 Network / Firebase console 狀態。

### 主線專案銜接

這一天接到主線專案的「EJS 是 server-side template engine，可在伺服器產生 HTML」部分。

具體流程：

1. 先在 $(System.Collections.Hashtable.files.Split('、')[0]) 建立或檢查當天需要的入口。
2. 把當天概念接進 Todo API / 小工具主線，而不是只寫孤立範例。
3. 執行對應的 node、npm、curl、瀏覽器或 Firebase console 驗證。

具體檢查項目：

- 檔案位置是否符合這一天的責任分層。
- 執行後是否能在 console、API response、瀏覽器或 Firebase 看到結果。
- 錯誤情境是否有明確訊息，而不是安靜失敗。

### 當天做完後檢查

- 能用自己的話說出「這一天解決什麼問題」。
- 能指出輸入、處理層與輸出位置。
- 能重跑範例並得到穩定結果。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 入口 | src/app.js | 接收需求或啟動當天流程 |
| 邏輯 | src/app.js、views/todos.ejs、src/routes/pages.js | 放置當天主要 Node.js / Firebase / Express 概念 |
| 驗證 | terminal、browser、Firebase console、curl | 確認結果、錯誤與資料狀態 |

### 完整實作流程、範例與注意事項

完整流程：

1. 建立或確認當天需要的檔案與資料夾。
2. 寫入這一天的最小可驗證片段。
3. 從入口檔、npm script、HTTP request 或瀏覽器事件觸發流程。
4. 檢查 console、HTTP response、畫面或 Firebase console。
5. 故意測一個錯誤情境，確認自己知道第一步要查哪裡。

範例與說明：

`	ext
app.set("view engine", "ejs");`napp.get("/todos-page", async (req, res) => {`n  res.render("todos", { todos: await listTodos() });`n});
`

注意事項：

- 不要只看程式碼能不能跑，要看它是否放在對的責任位置。
- 如果範例涉及資料寫入，先用測試資料庫或練習節點，不要直接操作正式資料。
- 如果結果不同，先確認 Node 版本、工作目錄、套件安裝、環境變數與 API endpoint。

### 如果結果和預期不同

1. 先複製完整錯誤訊息，不要只看最後一行。
2. 確認目前終端機所在目錄是不是專案根目錄。
3. 檢查檔案名稱大小寫、相對路徑、package.json scripts 與套件版本。
4. 若是 Firebase 或 HTTP，檢查 Network、權限規則、資料路徑與 status code。

### 負面例子 / 錯誤用法

錯誤做法：把所有流程都塞進 index.js，或看到錯誤就直接改成另一段網路範例，沒有先判斷責任層。

問題：

- 後續很難測試與重用。
- route、資料庫、畫面與設定互相耦合。
- 新人接手時不知道錯誤應該查哪個檔案。

修正方向：依照當天主題拆到環境設定、工具模組、資料存取、route、middleware、static 或 template 對應位置。

### 小練習

把今天的範例改成另一個小需求：新增 
otes 或 ookmarks 功能，並寫下輸入、處理層、輸出與驗證方式。

### Junior 常見誤解

- 以為能跑就代表設計正確；其實還要看放置位置與責任邊界。
- 以為範例中的簡化寫法可以直接上正式環境；正式環境還要補安全性、錯誤處理與測試。
- 以為錯誤都在程式碼；很多時候是工作目錄、版本、權限或資料路徑錯。

### 一句話總結

EJS 是 server-side template engine，可在伺服器產生 HTML，它讓 Node.js 主線專案多了一塊可以被驗證、維護與交付的能力。
## 全系列學習收束

這個系列的學習順序很適合 junior：先理解 Node.js 是什麼，再學 npm、模組、HTTP server、路徑處理、版本管理，接著用 Firebase 練 CRUD 與即時資料，最後用 Express、middleware、static 與 EJS 把專案整理成更像真實 Web app 的形狀。

實務上請記得：原文範例是入門階段，正式專案還要補測試、驗證、錯誤處理、權限、環境變數管理與部署流程。學習時的重點不是一次背完所有 API，而是每次接到需求都能判斷：入口在哪裡、資料怎麼流、應該改哪一層、交付前怎麼證明它真的可以用。
