# PHP 框架 Laravel 自學挑戰：2026 逐日實戰整理

來源系列：
- <https://ithelp.ithome.com.tw/m/users/20168986/ironman/7530>

整理日期：2026-05-04

這份筆記依照 iT 邦幫忙鐵人賽「PHP框架-Laravel自學挑戰」30 天逐日整理，並補上 2026 年仍實用的 Laravel 11/12、PHP 8.3/8.4、Vite、Livewire 3、Sanctum、GitHub REST API 與測試觀念。每一天都獨立包含學習目標、實作流程、範例、錯誤用法與練習，讀到哪一天就能練到哪一天。

## 這份筆記怎麼讀

Laravel 可以先想成一個分工清楚的 Web 開發工作台：

- `routes/`：決定 URL 入口。
- `Controller`：接住 request，協調流程。
- `Model` / Eloquent：和資料庫溝通。
- `Migration`：管理資料表版本。
- `Blade` / Livewire：產生畫面與互動。
- Middleware / Policy：處理登入、權限與流程保護。
- Service / Job：放外部 API 或比較長的工作。

每一天的範例都不是 production-ready，而是 junior 可以照著理解的最小完整範圍。

## 來源清單

| Day | 原文主題 |
| --- | --- |
| 1 | PHP 框架 Laravel 自學之旅開始 |
| 2 | Laravel 框架特色與純 PHP 差異比較 |
| 3 | 初次啟動 Laravel 專案，程式碼架構上 |
| 4 | 初次啟動 Laravel 專案，程式碼架構下 |
| 5 | Routes 和 Controllers |
| 6 | Blade 模板 1 |
| 7 | Blade 模板 2 |
| 8 | Blade 模板 3 |
| 9 | Controller 發送 GET Request |
| 10 | Controller 發送 POST Request |
| 11 | Eloquent ORM |
| 12 | Migration |
| 13 | Livewire 初認識 |
| 14 | Livewire Form 上 |
| 15 | Livewire Form 下 |
| 16 | Resume builder 專案發想與架構 |
| 17 | 登入畫面與功能上 |
| 18 | 登入畫面與功能下 |
| 19 | 登出功能與 navbar |
| 20 | 建立填寫表單 |
| 21 | 顯示 resume list |
| 22 | 顯示單筆 resume |
| 23 | 編輯 resume 上 |
| 24 | 編輯 resume 下 |
| 25 | 刪除 resume |
| 26 | 更新 GitHub README 上 |
| 27 | OAuth Token and Scope |
| 28 | PUT request 更新 README |
| 29 | 登入狀態檢查與程式碼優化 |
| 30 | 完賽心得與下一步 |

## 主線專案：Resume Builder 實戰累積地圖

這份 Laravel 筆記和 CSharp MVC 筆記最大的學習差異，應該補在「同一個專案每天往前長一點」。所以後續閱讀時，不要把每一天當成孤立語法；請把它們都接到同一個 `resume-builder` 專案。

### 專案最終會長成什麼

使用者可以註冊、登入、建立自己的履歷、查看履歷列表、編輯、刪除，最後把履歷轉成 Markdown 並更新到 GitHub README。這條主線會讓 Laravel 的 route、controller、Blade、Eloquent、migration、Livewire、auth、policy、service、外部 API 全部接在一起。

### 需要的檔案地圖

| 類型 | 位置 | 負責的事情 |
| --- | --- | --- |
| Route | `routes/web.php` | 定義 resume builder 的 URL 入口 |
| Controller | `app/Http/Controllers/ResumeController.php` | 處理 CRUD request |
| Publish Controller | `app/Http/Controllers/PublishResumeController.php` | 觸發 GitHub README 發布 |
| Model | `app/Models/Resume.php` | 對應 `resumes` 資料表 |
| Migration | `database/migrations/*create_resumes_table.php` | 建立履歷資料表 |
| Policy | `app/Policies/ResumePolicy.php` | 限制只能操作自己的履歷 |
| Formatter | `app/Support/ResumeMarkdownFormatter.php` | 把履歷資料轉成 Markdown |
| API Service | `app/Services/GitHubReadmeService.php` | 封裝 GitHub Contents API |
| Blade Layout | `resources/views/layouts/app.blade.php` | 共用版型與 navbar |
| Resume Views | `resources/views/resumes/*.blade.php` | list/create/show/edit 畫面 |
| Livewire Component | `app/Livewire/ResumeForm.php` | 互動式表單練習 |
| Tests | `tests/Feature/ResumeTest.php` | 驗證 CRUD 與權限 |

### 30 天交付物地圖

| Day | 當天交付物 | 做完後應該看得到 |
| --- | --- | --- |
| 1 | 建立 Laravel 專案 | Laravel 首頁可開啟 |
| 2 | 建立第一個 route/controller/view | `/home` 顯示頁面 |
| 3 | 找到主要目錄並建立 Resume model | `app/Models/Resume.php` 出現 |
| 4 | 建立 GitHub API 設定入口 | `config('services.github.api_url')` 可讀 |
| 5 | 建立 resume resource route | `php artisan route:list` 有 resumes |
| 6 | 建立共用 layout | 履歷頁套用共用 navbar |
| 7 | 抽出 resume card component | 列表頁重用卡片 UI |
| 8 | 建立安全 Blade form | 表單有 CSRF、old input、error |
| 9 | 建立 GitHub GET service | 可查 GitHub user JSON |
| 10 | 建立通知 POST 範例 | POST 失敗會被捕捉 |
| 11 | 建立 Resume Eloquent model | 可用 model 建立資料 |
| 12 | 建立 resumes migration | 資料庫有 `resumes` 表 |
| 13 | 建立 Livewire counter | 點擊後畫面更新 |
| 14 | 建立 Livewire 表單狀態 | 輸入可綁定並驗證 |
| 15 | Livewire 表單寫入資料庫 | 儲存後回列表 |
| 16 | 完成 MVP 與資料模型設計 | 有 user story 與資料表規劃 |
| 17 | 安裝 Breeze | `/login`、`/register` 可用 |
| 18 | 用 auth middleware 保護 resumes | 未登入會導到 login |
| 19 | Navbar 顯示登入/登出 | 登入者可 POST logout |
| 20 | 完成 create/store | 可新增自己的履歷 |
| 21 | 完成 index | 只看得到自己的履歷列表 |
| 22 | 完成 show + policy | 不能看別人的履歷 |
| 23 | 完成 edit form | 欄位有預填資料 |
| 24 | 完成 update | 可修改並回到 show |
| 25 | 完成 destroy | 可刪除自己的履歷 |
| 26 | 完成 Markdown formatter | 履歷可輸出 Markdown |
| 27 | 完成 token 設定 | token 不進 git，權限最小化 |
| 28 | 完成 GitHub PUT service | 可用 SHA 更新 README |
| 29 | 重構 middleware/policy/service | Controller 變薄 |
| 30 | 補 README、測試與收尾 | 專案可被重新啟動與檢查 |

### 主線端到端流程

1. 使用者進入首頁，透過 Breeze 註冊帳號。
2. 登入後進入 `/resumes`，此 route 受到 `auth` middleware 保護。
3. 使用者點「新增履歷」，進入 create form。
4. 表單送出到 `ResumeController@store`，controller 驗證資料。
5. 系統透過 `$request->user()->resumes()->create($data)` 建立履歷，避免使用者偽造 `user_id`。
6. 使用者回到列表，只看到自己的履歷。
7. 使用者點進 show/edit/delete，每個單筆操作都經過 `ResumePolicy`。
8. 使用者點「發布到 GitHub」，controller 呼叫 formatter 產生 Markdown。
9. `GitHubReadmeService` 先 GET README 取得 SHA，再 PUT 新內容。
10. 成功後 GitHub repository 產生一個 commit，系統顯示發布成功。

### 主線做完後檢查

- `php artisan route:list --name=resumes` 看得到 CRUD routes。
- 未登入開 `/resumes` 會被導到 login。
- A 使用者不能看、改、刪 B 使用者的履歷。
- 建立、編輯、刪除都只使用 validated data。
- GitHub token 只存在 `.env` 或正式環境 secret。
- GitHub PUT 流程有處理 401、403、409、timeout。
- `php artisan test` 至少涵蓋 resume CRUD 與權限。

---

## Day 1：PHP 框架 Laravel 自學之旅開始

### 這篇文章主要在講什麼

開始 Laravel 自學旅程，先理解為什麼 PHP 專案需要框架，以及 Laravel 在 Web 專案中扮演的角色。

### 為什麼需要這個概念

純 PHP 可以完成網站，但專案一大，路由、資料庫、安全、畫面組織會越來越亂。Laravel 提供一套慣例，讓團隊知道程式該放哪裡。

### 學完這篇你應該會做到什麼

你應該能建立 Laravel 專案，知道 Composer、Artisan、`.env`、Vite 各自負責什麼。

### 核心重點

- Composer 管 PHP 套件，npm/Vite 管前端資產。
- Artisan 是 Laravel 的 CLI 工具。
- `.env` 放環境設定，不該 commit 真實密碼。
- Laravel 不是取代 PHP，而是把 PHP Web 開發整理成框架慣例。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「專案啟動」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 專案啟動 | 建立 resume-builder Laravel 專案 | 首頁可開啟，Artisan 與 Vite 可啟動 |

當天接到主線專案的流程：

1. 建立專案並確認 PHP、Composer、Node 都能運作。
2. 啟動 php artisan serve，確認 Laravel 首頁。
3. 啟動 npm run dev，確認前端資產流程。

### 當天做完後檢查

- php artisan serve 可開啟本機站台。
- .env 已存在且有 app key。
- npm run dev 沒有報錯。
### 範例範圍地圖

| 部分 | 位置 / 指令 | 負責的事情 |
| --- | --- | --- |
| 建立專案 | `composer create-project laravel/laravel resume-builder` | 產生 Laravel 專案 |
| 啟動後端 | `php artisan serve` | 啟動開發伺服器 |
| 啟動前端 | `npm run dev` | 編譯 Vite 資產 |

### 完整實作流程、範例與注意事項

1. 確認已安裝 PHP 8.3 以上、Composer、Node.js LTS。
2. 執行 `composer create-project laravel/laravel resume-builder`。
3. 進入專案後執行 `cp .env.example .env` 與 `php artisan key:generate`。
4. 執行 `php artisan serve`。
5. 另一個終端執行 `npm install`、`npm run dev`。
6. 開啟 `http://127.0.0.1:8000` 確認首頁正常。

```powershell
# 範例用途：建立並啟動 Laravel 練習專案。
# 主要輸入：resume-builder 是資料夾名稱，可換成自己的專案名。
# 預期結果：瀏覽器可看到 Laravel 首頁。
composer create-project laravel/laravel resume-builder
cd resume-builder
php artisan key:generate
php artisan serve
```

### 如果結果和預期不同

- 找不到 `composer`：確認 Composer 已加入 PATH。
- `No application encryption key`：執行 `php artisan key:generate`。
- Vite 資產載入失敗：確認 `npm install` 與 `npm run dev` 有跑。

### 負面例子 / 錯誤用法

錯誤做法：把資料庫密碼直接寫在 PHP 檔。
問題：憑證容易被 commit，也難以在本機、測試、正式環境切換。
修正方向：放在 `.env`，程式透過 `config()` 讀取。

### 小練習

建立一個 `hello-laravel` 專案，啟動後確認首頁與 Vite 都正常。

### Junior 常見誤解

Laravel 不是「比較潮的 PHP 語法」，而是一整套專案組織方式。

### 一句話總結

Laravel 的第一步不是背語法，而是建立「框架幫我整理專案邊界」的心智模型。

---

## Day 2：Laravel 框架特色與純 PHP 差異比較

### 這篇文章主要在講什麼

比較 Laravel 與純 PHP 的差異，理解框架提供的路由、MVC、ORM、安全與工具鏈。

### 為什麼需要這個概念

如果不知道框架解決什麼問題，就容易把 Laravel 寫成大型單檔 PHP，只是換了資料夾而已。

### 學完這篇你應該會做到什麼

你應該能說明 request 從 URL 進來後，如何經過 route、controller、model、view。

### 核心重點

- 純 PHP 常常一頁同時處理 SQL、HTML、權限與表單。
- Laravel 把入口、流程、資料與畫面拆開。
- ORM 可降低手寫 SQL 的重複，但仍要理解 SQL。
- Laravel 內建 CSRF、validation、escaping 等安全工具。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「入口流程」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 入口流程 | 建立第一組 route、controller、view | /home 可透過 controller 顯示 Blade |

當天接到主線專案的流程：

1. 在 routes/web.php 建立 /home。
2. 讓 route 指向 PageController@home。
3. Controller 傳資料給 Blade，而不是直接 echo HTML。

### 當天做完後檢查

- php artisan route:list 看得到 /home。
- Blade 能顯示 controller 傳入的 title。
- route 沒有塞大量商業邏輯。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Route | `routes/web.php` | URL 對應 controller |
| Controller | `app/Http/Controllers/PageController.php` | 協調流程 |
| View | `resources/views/welcome.blade.php` | 顯示 HTML |

### 完整實作流程、範例與注意事項

1. 建立 controller：`php artisan make:controller PageController`。
2. 在 controller 放一個 `home()` 方法。
3. 在 `routes/web.php` 指向 controller。
4. 建立 Blade view。
5. 用瀏覽器確認 `/home` 能看到內容。

```php
<?php

// routes/web.php
// 範例用途：示範 Laravel 把 URL 入口集中管理。
// 主要輸入：GET /home。
// 回傳結果：呼叫 PageController::home 並產生 HTML。

use App\Http\Controllers\PageController;
use Illuminate\Support\Facades\Route;

Route::get('/home', [PageController::class, 'home'])->name('home');
```

```php
<?php

namespace App\Http\Controllers;

use Illuminate\View\View;

class PageController extends Controller
{
    public function home(): View
    {
        // Controller：只準備資料，不直接 echo HTML。
        return view('home', [
            'title' => '我的 Laravel 首頁',
        ]);
    }
}
```

### 如果結果和預期不同

- 404：確認 route 是否註冊，執行 `php artisan route:list`。
- View not found：確認檔案在 `resources/views/home.blade.php`。

### 負面例子 / 錯誤用法

錯誤做法：在 route closure 裡塞大量商業邏輯與 SQL。
問題：之後測試、重用、權限控制都會變困難。
修正方向：route 保持薄，流程放 controller/service。

### 小練習

新增 `/about` route 與 `about.blade.php`，練習命名 route。

### Junior 常見誤解

MVC 不是多建幾個檔案而已，重點是讓責任分開。

### 一句話總結

Laravel 和純 PHP 最大差異是它先幫你畫出專案分工線。

---

## Day 3：初次啟動 Laravel 專案，程式碼架構上

### 這篇文章主要在講什麼

認識 Laravel 專案主要目錄，特別是 `app/`、`routes/`、`resources/`、`database/`。

### 為什麼需要這個概念

找不到程式放哪裡，會讓你每次新增功能都亂猜，最後破壞框架慣例。

### 學完這篇你應該會做到什麼

你應該能知道新增頁面、資料模型、資料表變更、Blade 畫面要去哪裡改。

### 核心重點

- `app/` 放主要 PHP 程式。
- `routes/` 放入口規則。
- `resources/views` 放 Blade。
- `database/migrations` 放資料庫結構版本。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「專案結構」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 專案結構 | 建立 Resume model 與基本資料夾認知 | 知道履歷功能會分散在哪些 Laravel 目錄 |

當天接到主線專案的流程：

1. 建立 Resume model。
2. 找到 migration、view、controller 的放置位置。
3. 把後續履歷 CRUD 檔案位置先對上。

### 當天做完後檢查

- app/Models/Resume.php 存在。
- 知道 resources/views/resumes 是履歷畫面位置。
- 知道 migration 會進 database/migrations。
### 範例範圍地圖

| 需求 | 位置 |
| --- | --- |
| 新增履歷頁 URL | `routes/web.php` |
| 新增履歷資料模型 | `app/Models/Resume.php` |
| 新增資料表 | `database/migrations` |
| 新增畫面 | `resources/views/resumes` |

### 完整實作流程、範例與注意事項

1. 執行 `php artisan make:model Resume -m`。
2. 查看 `app/Models/Resume.php`。
3. 查看新 migration。
4. 建立 `resources/views/resumes/index.blade.php`。
5. 新增 route 回傳 view。

```php
<?php

// routes/web.php
// 範例用途：快速確認 view 放置位置。
// 主要輸入：GET /resumes。
// 回傳結果：顯示 resources/views/resumes/index.blade.php。

Route::view('/resumes', 'resumes.index')->name('resumes.index');
```

### 如果結果和預期不同

- 找不到 view：Blade 路徑用點號對應資料夾，例如 `resumes.index`。
- Model 找不到：確認 namespace 是 `App\Models`。

### 負面例子 / 錯誤用法

錯誤做法：把所有 PHP class 都放在專案根目錄。
問題：autoload、命名空間與團隊慣例會混亂。
修正方向：依 Laravel 預設目錄放置。

### 小練習

建立 `Project` model 與 migration，確認它們分別出現在正確目錄。

### Junior 常見誤解

目錄不是裝飾，是 Laravel 用來讓框架、工具與團隊理解你的程式。

### 一句話總結

先熟悉目錄，才不會在 Laravel 裡迷路。

---

## Day 4：初次啟動 Laravel 專案，程式碼架構下

### 這篇文章主要在講什麼

補齊 `config/`、`.env`、`public/`、`storage/`、`bootstrap/`、Artisan 等專案結構。

### 為什麼需要這個概念

Laravel 專案不只有 app code；設定、公開入口、上傳檔案、快取與 log 都會影響真實部署。

### 學完這篇你應該會做到什麼

你應該知道設定在哪裡改、log 到哪裡看、公開檔案從哪裡進入。

### 核心重點

- `.env` 是環境設定來源。
- `config/*.php` 是應用程式讀取設定的正式位置。
- `public/index.php` 是 Web server 入口。
- `storage/logs/laravel.log` 是排錯第一站。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「環境設定」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 環境設定 | 建立 GitHub API 設定入口 | GitHub API URL 能從 config 讀取 |

當天接到主線專案的流程：

1. 在 .env 放外部服務設定。
2. 在 config/services.php 統一整理。
3. 程式只讀 config，不散落讀 env。

### 當天做完後檢查

- services.github.api_url 可讀。
- 真實 token 沒有寫進程式碼。
- 改設定後知道要清 config cache。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| 環境變數 | `.env` | 本機環境差異 |
| 設定檔 | `config/services.php` | 統一讀取外部服務設定 |
| Log | `storage/logs/laravel.log` | 排查錯誤 |

### 完整實作流程、範例與注意事項

1. 在 `.env` 加上 `GITHUB_API_URL=https://api.github.com`。
2. 在 `config/services.php` 加上 github 設定。
3. 用 `config('services.github.api_url')` 讀取。
4. 若改了 config cache，執行 `php artisan config:clear`。

```php
<?php

// config/services.php
// 範例用途：把外部服務設定集中，不讓程式到處讀 env()。
// 主要輸入：.env 裡的 GITHUB_API_URL。
// 回傳結果：程式可用 config('services.github.api_url') 取得值。

return [
    'github' => [
        'api_url' => env('GITHUB_API_URL', 'https://api.github.com'),
    ],
];
```

### 如果結果和預期不同

- 設定改了沒生效：執行 `php artisan config:clear`。
- 500 error：先看 `storage/logs/laravel.log`。

### 負面例子 / 錯誤用法

錯誤做法：在 application code 到處呼叫 `env()`。
問題：config cache 後可能讀不到預期值。
修正方向：在 config 檔讀 `env()`，程式讀 `config()`。

### 小練習

新增一個 `APP_SUPPORT_EMAIL` 設定，並在 Blade 顯示。

### Junior 常見誤解

`.env` 不是萬用資料庫，真正給程式讀的設定應該整理進 `config/`。

### 一句話總結

Laravel 設定的好習慣是 `.env` 管差異，`config()` 管讀取。

---

## Day 5：練習 Routes 和 Controllers

### 這篇文章主要在講什麼

學習如何用 route 對應 controller，建立清楚的 HTTP 入口。

### 為什麼需要這個概念

Route 是使用者進入系統的門。門口亂了，後面的功能也會變難維護。

### 學完這篇你應該會做到什麼

你應該能建立 resource controller，並用 route model binding 顯示單筆資料。

### 核心重點

- `Route::get()` 適合簡單頁面。
- `Route::resource()` 適合 CRUD。
- Route name 讓 redirect 與 link 不依賴硬編 URL。
- Route model binding 可以自動查資料與處理 404。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「CRUD 入口」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| CRUD 入口 | 建立 resume resource route 與 controller | resumes routes 出現在 route list |

當天接到主線專案的流程：

1. 建立 ResumeController。
2. 註冊 resource route。
3. 先完成 index 與 show 的入口。

### 當天做完後檢查

- php artisan route:list --name=resumes 有結果。
- route 使用 named route。
- controller method 保持薄。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Route | `routes/web.php` | 註冊 resume CRUD |
| Controller | `ResumeController` | 處理列表與單筆頁 |
| View | `resumes/index.blade.php` | 顯示資料 |

### 完整實作流程、範例與注意事項

1. 執行 `php artisan make:controller ResumeController --resource`。
2. 在 `routes/web.php` 加 `Route::resource('resumes', ResumeController::class)`。
3. 實作 `index()` 與 `show(Resume $resume)`。
4. 執行 `php artisan route:list --name=resumes` 檢查。

```php
<?php

namespace App\Http\Controllers;

use App\Models\Resume;
use Illuminate\View\View;

class ResumeController extends Controller
{
    public function index(): View
    {
        // 範例用途：顯示履歷列表。
        // 回傳結果：把分頁資料交給 Blade。
        return view('resumes.index', [
            'resumes' => Resume::query()->latest()->paginate(10),
        ]);
    }

    public function show(Resume $resume): View
    {
        // Route model binding：URL 的 {resume} 會自動變成 Resume 物件。
        return view('resumes.show', ['resume' => $resume]);
    }
}
```

### 如果結果和預期不同

- `Missing required parameter`：產生 route link 時沒有傳入 `$resume`。
- 404：資料不存在或 route pattern 不符。

### 負面例子 / 錯誤用法

錯誤做法：在 Blade 寫死 `/resumes/1`。
問題：路由改名或前綴改變時會壞。
修正方向：使用 `route('resumes.show', $resume)`。

### 小練習

替 `projects` 建一組 resource route，只開 `index`、`show`。

### Junior 常見誤解

Controller 不是越大越好；它應該協調流程，不該吃下整個系統。

### 一句話總結

Route 管入口，Controller 管流程，這條線先畫清楚。

---

## Day 6：Blade 模板 1

### 這篇文章主要在講什麼

認識 Blade 基本語法，包含變數輸出、layout、section。

### 為什麼需要這個概念

沒有模板繼承時，每頁都會重複 HTML head、navbar、script，維護成本很高。

### 學完這篇你應該會做到什麼

你應該能建立共用 layout，讓不同頁面只填自己的內容。

### 核心重點

- `{{ }}` 會 escape HTML。
- `@extends` 使用共用 layout。
- `@section` 填入 layout 的區塊。
- `@yield` 是 layout 預留的插槽。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「共用版型」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 共用版型 | 建立 layouts/app.blade.php | 履歷頁共用同一個 navbar 與 layout |

當天接到主線專案的流程：

1. 建立共用 layout。
2. 在履歷頁用 @extends 套版。
3. 把共用 navbar 放在 layout。

### 當天做完後檢查

- 改 navbar 只需改一個檔案。
- 子頁面有 content section。
- 沒有每頁複製完整 HTML。
### 範例範圍地圖

| View | 位置 | 負責的事情 |
| --- | --- | --- |
| Layout | `resources/views/layouts/app.blade.php` | 共用外框 |
| Page | `resources/views/resumes/index.blade.php` | 履歷列表內容 |

### 完整實作流程、範例與注意事項

1. 建立 `resources/views/layouts/app.blade.php`。
2. 放入 HTML 骨架與 `@yield('content')`。
3. 建立 `resumes/index.blade.php`。
4. 用 `@extends('layouts.app')` 套用 layout。

```blade
{{-- resources/views/layouts/app.blade.php --}}
{{-- 範例用途：提供所有頁面的共用 HTML 骨架。 --}}
<!doctype html>
<html lang="zh-Hant">
<head>
    <meta charset="utf-8">
    <title>{{ $title ?? 'Resume Builder' }}</title>
</head>
<body>
    <nav><a href="{{ route('resumes.index') }}">履歷列表</a></nav>
    <main>@yield('content')</main>
</body>
</html>
```

```blade
{{-- resources/views/resumes/index.blade.php --}}
@extends('layouts.app')

@section('content')
    <h1>履歷列表</h1>
@endsection
```

### 如果結果和預期不同

- 畫面空白：確認子頁有 `@section('content')`。
- 變數未定義：controller 是否傳入該變數。

### 負面例子 / 錯誤用法

錯誤做法：每個頁面都複製完整 HTML。
問題：改 navbar 要改很多檔。
修正方向：抽 layout。

### 小練習

在 layout 加 footer，確認所有套用 layout 的頁面都出現。

### Junior 常見誤解

Blade layout 不是前端框架，它是在伺服器端組出 HTML。

### 一句話總結

Blade layout 讓共用畫面只寫一次。

---

## Day 7：Blade 模板 2

### 這篇文章主要在講什麼

延伸 Blade：partial、component、slot 與資料傳遞。

### 為什麼需要這個概念

列表卡片、警告訊息、按鈕等 UI 若一直複製，會讓畫面不一致。

### 學完這篇你應該會做到什麼

你應該能把可重複的履歷卡片抽成 Blade component。

### 核心重點

- `@include` 適合簡單 partial。
- Blade component 適合可重用 UI。
- `$slot` 放 component 包住的內容。
- props 讓 component 接收資料。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「可重用 UI」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 可重用 UI | 建立 resume card Blade component | 列表頁用 component 顯示每份履歷 |

當天接到主線專案的流程：

1. 建立 resume-card component。
2. 由 index view 傳入 resume。
3. Component 只顯示資料，不查資料。

### 當天做完後檢查

- 列表頁有 resume card component。
- component 沒有 Eloquent 查詢。
- 連結使用 named route。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Component | `resources/views/components/resume-card.blade.php` | 顯示單張卡 |
| 使用端 | `resumes/index.blade.php` | 呼叫 component |

### 完整實作流程、範例與注意事項

1. 建立 `components/resume-card.blade.php`。
2. 在 index view 用 `<x-resume-card :resume="$resume" />`。
3. 確認 component 只負責顯示，不查資料庫。

```blade
{{-- resources/views/components/resume-card.blade.php --}}
{{-- 範例用途：重用履歷卡片 UI。主要輸入：$resume。 --}}
<article>
    <h2>{{ $resume->title }}</h2>
    <p>{{ $resume->summary }}</p>
    <a href="{{ route('resumes.show', $resume) }}">查看</a>
</article>
```

```blade
{{-- resources/views/resumes/index.blade.php --}}
@foreach ($resumes as $resume)
    <x-resume-card :resume="$resume" />
@endforeach
```

### 如果結果和預期不同

- Component 找不到：檔名要用 kebab-case，例如 `resume-card.blade.php` 對應 `<x-resume-card>`。

### 負面例子 / 錯誤用法

錯誤做法：component 內直接 `Resume::latest()->get()`。
問題：UI 元件偷偷查資料，會讓效能與資料流難追。
修正方向：資料由 controller 傳入。

### 小練習

做一個 `<x-alert>` component，可顯示成功或錯誤訊息。

### Junior 常見誤解

Component 不代表可以把所有邏輯塞進去，它首先是重用畫面。

### 一句話總結

Blade component 能減少重複，但資料來源仍要清楚。

---

## Day 8：Blade 模板 3

### 這篇文章主要在講什麼

學習 Blade 的條件、迴圈、表單與 escaping。

### 為什麼需要這個概念

真實畫面常需要依登入狀態、資料是否存在、驗證錯誤顯示不同內容。

### 學完這篇你應該會做到什麼

你應該能建立安全的 Laravel 表單，顯示驗證錯誤與舊輸入。

### 核心重點

- `@csrf` 保護表單。
- `old()` 顯示上次輸入。
- `@error` 顯示驗證錯誤。
- `@forelse` 同時處理清單與空資料。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「表單基礎」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 表單基礎 | 建立安全的 create form | 表單有 CSRF、old input、error message |

當天接到主線專案的流程：

1. 建立 resumes/create.blade.php。
2. 加上 CSRF 與 old input。
3. 顯示驗證錯誤。

### 當天做完後檢查

- 缺 title 會顯示錯誤。
- 驗證失敗後輸入值保留。
- 沒有用未 escape HTML 顯示使用者輸入。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| 表單 | `resumes/create.blade.php` | 收集輸入 |
| Controller | `ResumeController@store` | 驗證與保存 |

### 完整實作流程、範例與注意事項

1. 建立 create view。
2. 表單 method 用 POST，action 指向 `resumes.store`。
3. 加上 `@csrf`。
4. input value 用 `old('title')`。
5. 用 `@error('title')` 顯示錯誤。

```blade
<form method="POST" action="{{ route('resumes.store') }}">
    @csrf

    <label>
        標題
        <input name="title" value="{{ old('title') }}">
    </label>
    @error('title')
        <p>{{ $message }}</p>
    @enderror

    <button type="submit">儲存</button>
</form>
```

### 如果結果和預期不同

- 419 Page Expired：缺 `@csrf`。
- 驗證失敗後資料消失：沒有用 `old()`。

### 負面例子 / 錯誤用法

錯誤做法：用 `{!! $resume->summary !!}` 顯示使用者輸入。
問題：可能造成 XSS。
修正方向：預設用 `{{ $resume->summary }}`。

### 小練習

替 summary 加 textarea，驗證錯誤後保留舊值。

### Junior 常見誤解

Blade 的 `{{ }}` 不是單純 echo，它預設幫你做 HTML escaping。

### 一句話總結

表單最重要的是安全、驗證、錯誤回饋三件事一起做。

---

## Day 9：利用 Controller 發送 API GET Request

### 這篇文章主要在講什麼

在 Laravel 內呼叫外部 API 的 GET request。

### 為什麼需要這個概念

很多系統都要串第三方服務，例如 GitHub、天氣、付款、通知。外部 API 不穩定，所以不能把它當成本機 function。

### 學完這篇你應該會做到什麼

你應該能用 Laravel HTTP client 發 GET，並處理 timeout 與失敗。

### 核心重點

- 使用 `Http::get()`。
- 設定 `timeout()`。
- 檢查 `successful()` 或 `failed()`。
- API URL 與 token 放 config。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「外部 GET API」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 外部 GET API | 建立 GitHub GET service | 可查 GitHub 使用者公開資料 |

當天接到主線專案的流程：

1. 建立 GitHubProfileService。
2. 用 Laravel HTTP client 發 GET。
3. 設定 timeout 並處理 failed response。

### 當天做完後檢查

- API 失敗不會讓頁面無提示爆掉。
- service 可獨立測試。
- controller 不直接塞 API 細節。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Service | `app/Services/GitHubProfileService.php` | 呼叫 GitHub API |
| Controller | `ProfileController` | 顯示結果 |

### 完整實作流程、範例與注意事項

1. 建立 service class。
2. 用 `Http::timeout(10)->get(...)` 呼叫 API。
3. 失敗時丟出例外或回傳錯誤狀態。
4. Controller 呼叫 service，再把結果傳給 view。

```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Http;

class GitHubProfileService
{
    public function getUser(string $username): array
    {
        // 範例用途：查詢 GitHub 使用者公開資料。
        // 主要輸入：$username 來自路由或表單。
        // 回傳結果：GitHub JSON 轉成 array；失敗時丟例外。
        $response = Http::acceptJson()
            ->timeout(10)
            ->get("https://api.github.com/users/{$username}");

        $response->throw();

        return $response->json();
    }
}
```

### 如果結果和預期不同

- timeout：確認網路或 API 狀態，必要時 retry。
- 404：username 不存在。
- 403：GitHub rate limit 或 token 權限問題。

### 負面例子 / 錯誤用法

錯誤做法：controller 裡直接呼叫 API 並假設一定成功。
問題：API 掛掉時整頁 500，且難測試。
修正方向：抽 service，處理失敗狀態。

### 小練習

建立一頁輸入 GitHub username，顯示 avatar URL 與 public repos。

### Junior 常見誤解

外部 API 不是你的資料庫，永遠要預期它會慢、會錯、會限流。

### 一句話總結

GET API 的基本功是 timeout、錯誤處理與清楚的 service 邊界。

---

## Day 10：利用 Controller 發送 API POST Request

### 這篇文章主要在講什麼

練習用 Laravel HTTP client 發送 POST request。

### 為什麼需要這個概念

POST 常用在建立資料、送表單、呼叫 webhook 或外部服務操作。

### 學完這篇你應該會做到什麼

你應該能送 JSON body，理解 header、body、status code。

### 核心重點

- `Http::post()` 預設可送 array 轉 JSON。
- `withToken()` 放 Bearer token。
- `throw()` 可把失敗 response 轉例外。
- 不要記錄敏感 body。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「外部 POST API」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 外部 POST API | 建立通知 POST service 範例 | POST body 經 validation/mapping 後送出 |

當天接到主線專案的流程：

1. 先驗證內部表單資料。
2. Mapping 成外部 API 要的 JSON。
3. 發送 POST 並處理 401/422/timeout。

### 當天做完後檢查

- 沒有把 request 全部轉發。
- token 從 config 讀取。
- 失敗時有可理解錯誤。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Request DTO | array / Form Request | 準備資料 |
| Service | `NotificationService` | 發送外部 API |
| Controller | 呼叫 service | 回應使用者 |

### 完整實作流程、範例與注意事項

1. 驗證表單輸入。
2. 把資料整理成 API 需要的 JSON。
3. 用 `Http::withToken()->post()`。
4. 根據 status code 回應成功或失敗。

```php
<?php

use Illuminate\Support\Facades\Http;

function sendResumePublishNotification(string $email, string $resumeTitle): void
{
    // 範例用途：通知外部服務履歷已發布。
    // 主要輸入：email 與 resumeTitle 來自已驗證資料。
    // 回傳結果 / 副作用：外部通知服務收到一筆通知 request。
    Http::withToken(config('services.notify.token'))
        ->acceptJson()
        ->timeout(10)
        ->post('https://notify.example.test/messages', [
            'to' => $email,
            'subject' => 'Resume published',
            'message' => "你的履歷 {$resumeTitle} 已發布。",
        ])
        ->throw();
}
```

### 如果結果和預期不同

- 422：送出的 JSON 欄位不符合 API 規格。
- 401：token 錯誤或過期。
- 500：外部服務錯，應顯示友善訊息而不是堆疊追蹤。

### 負面例子 / 錯誤用法

錯誤做法：把使用者輸入原封不動轉發給外部 API。
問題：可能送出多餘欄位或惡意內容。
修正方向：先 validation，再 mapping 成 API DTO。

### 小練習

把 POST request 包成 service，並在 controller 捕捉失敗例外。

### Junior 常見誤解

POST 不代表一定成功建立資料，要看 status code 與 response body。

### 一句話總結

POST API 要先整理資料，再送出，最後處理失敗。

---

## Day 11：與 Eloquent ORM 初認識

### 這篇文章主要在講什麼

認識 Laravel Eloquent ORM，用 Model 操作資料表。

### 為什麼需要這個概念

ORM 讓資料表和 PHP class 有對應關係，減少重複 SQL，但你仍然要理解查詢成本。

### 學完這篇你應該會做到什麼

你應該能建立 model、設定 fillable、查詢與建立資料。

### 核心重點

- Model 對應資料表。
- `fillable` 控制可批量寫入欄位。
- Query builder 可串接條件。
- Relationship 表達資料關係。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「資料模型」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 資料模型 | 設定 Resume Eloquent model | 可用 model 建立與查詢履歷 |

當天接到主線專案的流程：

1. 設定 fillable。
2. 定義 User 與 Resume 關聯。
3. 從登入者關聯建立資料。

### 當天做完後檢查

- 沒有 create request all。
- user_id 不是使用者表單傳入。
- tinker 可查到建立的資料。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Model | `app/Models/Resume.php` | 對應 resumes |
| Controller | `store()` | 建立資料 |

### 完整實作流程、範例與注意事項

1. 建立 `Resume` model。
2. 設定 `$fillable`。
3. 在 controller 用登入者關聯建立資料。
4. 用 tinker 查詢確認。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Resume extends Model
{
    // 範例用途：允許這些欄位被表單批量寫入。
    // 注意：user_id 通常由登入者決定，不讓使用者任意傳。
    protected $fillable = ['title', 'summary'];
}
```

```php
$resume = auth()->user()->resumes()->create([
    'title' => 'Junior Backend Resume',
    'summary' => '熟悉 Laravel 基礎 CRUD',
]);
```

### 如果結果和預期不同

- MassAssignmentException：沒有設定 fillable。
- 寫入 user_id 錯誤：確認 relationship 與登入者。

### 負面例子 / 錯誤用法

錯誤做法：`Resume::create($request->all())`。
問題：使用者可能塞入不該寫入的欄位。
修正方向：使用 `$request->validated()`。

### 小練習

建立 `Experience` model，讓一份 resume 有多筆 experience。

### Junior 常見誤解

ORM 不是不用學 SQL，而是讓常見 SQL 更有結構。

### 一句話總結

Eloquent 的重點是用 Model 管資料，但輸入邊界仍要守好。

---

## Day 12：Migration：資料庫結構管理

### 這篇文章主要在講什麼

用 migration 管理資料表建立、修改與 rollback。

### 為什麼需要這個概念

團隊不能靠口頭說「你資料庫加一欄」。Migration 讓資料庫結構跟著 git 版本走。

### 學完這篇你應該會做到什麼

你應該能建立 `resumes` 資料表並 rollback。

### 核心重點

- `up()` 定義套用變更。
- `down()` 定義回復變更。
- `php artisan migrate` 執行。
- `php artisan migrate:rollback` 回復上一批。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「資料庫版本」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 資料庫版本 | 建立 resumes migration | 資料庫有 resumes 表 |

當天接到主線專案的流程：

1. 定義 resumes 欄位。
2. 加上 user_id foreign key。
3. 執行 migrate 並確認資料表。

### 當天做完後檢查

- php artisan migrate 成功。
- rollback 能回復。
- 沒有手動改 DB 卻不寫 migration。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Migration | `database/migrations/...create_resumes_table.php` | 建表 |
| Database | `resumes` | 保存履歷 |

### 完整實作流程、範例與注意事項

1. 執行 `php artisan make:migration create_resumes_table`。
2. 在 migration 定義欄位。
3. 執行 `php artisan migrate`。
4. 用資料庫工具確認表存在。

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('resumes', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->string('title');
            $table->text('summary')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('resumes');
    }
};
```

### 如果結果和預期不同

- SQL error：確認資料庫連線與欄位型別。
- rollback 失敗：確認 `down()` 能反向操作。

### 負面例子 / 錯誤用法

錯誤做法：正式環境直接手動改資料表，但不寫 migration。
問題：其他環境不會同步，部署容易壞。
修正方向：所有 schema 變更都寫 migration。

### 小練習

替 `resumes` 加 `published_at` 欄位，寫一支新的 migration。

### Junior 常見誤解

Migration 不是備份資料，它是資料表結構版本。

### 一句話總結

Migration 讓資料庫結構變成可追蹤、可重播的程式碼。

---

## Day 13：Laravel Livewire 初認識

### 這篇文章主要在講什麼

認識 Livewire：用 PHP 寫互動式 UI，不必完整建立前端 SPA。

### 為什麼需要這個概念

很多後台表單只需要小互動，用 Vue/React 可能太重；Livewire 可用 Laravel 生態快速完成。

### 學完這篇你應該會做到什麼

你應該能建立 Livewire component 並在 Blade 中使用。

### 核心重點

- Livewire component 有 PHP class 與 Blade view。
- public property 會和前端同步。
- method 可由 `wire:click`、`wire:submit` 呼叫。
- 敏感資料不要放 public property。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「互動元件起點」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 互動元件起點 | 建立 Livewire counter | 點擊按鈕會觸發 PHP method 更新畫面 |

當天接到主線專案的流程：

1. 安裝或確認 Livewire。
2. 建立 Counter component。
3. 用 wire click 驗證互動。

### 當天做完後檢查

- Livewire scripts/styles 正常。
- public property 不含敏感資料。
- 互動失敗時知道查 browser console/network。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Component class | `app/Livewire/Counter.php` | 保存狀態與方法 |
| Component view | `resources/views/livewire/counter.blade.php` | 顯示與觸發 |

### 完整實作流程、範例與注意事項

1. 安裝 Livewire。
2. 執行 `php artisan make:livewire Counter`。
3. 在 class 建立 `$count` 與 `increment()`。
4. 在 Blade 使用 `<livewire:counter />`。

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public int $count = 0;

    public function increment(): void
    {
        // 範例用途：示範前端點擊觸發 PHP method。
        // 回傳結果 / 副作用：count 加一，畫面自動更新。
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

```blade
<button wire:click="increment">加一</button>
<p>{{ $count }}</p>
```

### 如果結果和預期不同

- 點擊沒反應：確認 layout 有 Livewire scripts/styles。
- 狀態不同步：確認 property 是 public。

### 負面例子 / 錯誤用法

錯誤做法：把 API token 放 public property。
問題：Livewire state 可能暴露到前端。
修正方向：敏感資料放 server-side config 或 private property。

### 小練習

做一個 `SearchBox` component，輸入關鍵字後顯示目前搜尋字串。

### Junior 常見誤解

Livewire 不是不用懂前端，而是把互動 request 包裝得更像 Laravel。

### 一句話總結

Livewire 適合 Laravel 後台互動，但資料暴露邊界要小心。

---

## Day 14：用 Livewire 撰寫 Form 上

### 這篇文章主要在講什麼

用 Livewire 建立表單狀態與即時驗證。

### 為什麼需要這個概念

表單是後台最常見的功能，Livewire 可讓輸入、錯誤訊息與送出流程都在同一個 component 中管理。

### 學完這篇你應該會做到什麼

你應該能用 `wire:model` 綁定輸入，並用 rules 驗證。

### 核心重點

- `wire:model` 綁定 property。
- `validate()` 執行 server-side validation。
- `.blur` 或 `.debounce` 可減少 request。
- 錯誤訊息用 `@error` 顯示。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「Livewire 表單狀態」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| Livewire 表單狀態 | 建立 ResumeForm 狀態與 rules | 輸入欄位可綁定並顯示驗證錯誤 |

當天接到主線專案的流程：

1. 建立 ResumeForm component。
2. 用 public properties 綁定欄位。
3. 定義 validation rules。

### 當天做完後檢查

- wire:model.blur 可同步資料。
- 錯誤訊息能對應欄位。
- 不只靠前端 maxlength。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Livewire class | `ResumeForm.php` | 表單狀態與驗證 |
| View | `resume-form.blade.php` | 輸入欄位 |

### 完整實作流程、範例與注意事項

1. 建立 `ResumeForm` component。
2. 宣告 `title` 與 `summary`。
3. 寫 rules。
4. view 中使用 `wire:model.blur`。

```php
public string $title = '';
public string $summary = '';

protected function rules(): array
{
    return [
        'title' => ['required', 'string', 'max:120'],
        'summary' => ['nullable', 'string', 'max:1000'],
    ];
}
```

```blade
<input wire:model.blur="title">
@error('title') <p>{{ $message }}</p> @enderror
```

### 如果結果和預期不同

- 每打字都發 request：改用 `.blur` 或 `.debounce.500ms`。
- 錯誤訊息不出現：確認欄位名稱和 rules 一樣。

### 負面例子 / 錯誤用法

錯誤做法：只在前端限制 maxlength，不做 server validation。
問題：使用者可繞過前端直接送 request。
修正方向：Livewire class 內一定要 validate。

### 小練習

替 title 加上最少 3 字元限制。

### Junior 常見誤解

`wire:model` 很方便，但不是取代 validation。

### 一句話總結

Livewire form 的核心是狀態、驗證、錯誤訊息三者一致。

---

## Day 15：用 Livewire 撰寫 Form 下

### 這篇文章主要在講什麼

完成 Livewire 表單送出、寫入資料庫與導頁。

### 為什麼需要這個概念

表單不是畫出 input 就結束，真正重要的是資料驗證後如何安全保存。

### 學完這篇你應該會做到什麼

你應該能在 Livewire `save()` 中建立 resume。

### 核心重點

- submit 用 `wire:submit`。
- 儲存前先 validate。
- 使用登入者關聯建立資料。
- 成功後 redirect 或顯示 flash message。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「Livewire 儲存流程」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| Livewire 儲存流程 | 在 save 寫入 resumes | 表單送出後建立履歷並回列表 |

當天接到主線專案的流程：

1. wire submit 呼叫 save。
2. save 裡先 validate。
3. 用登入者關聯建立 resume。

### 當天做完後檢查

- 未登入不會呼叫到 user resumes。
- Mass assignment 沒有錯。
- 成功後 redirect 正確。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Component | `ResumeForm` | 儲存資料 |
| Model | `Resume` | 寫入資料庫 |

### 完整實作流程、範例與注意事項

1. 在 view 包 `<form wire:submit="save">`。
2. 在 class 實作 `save()`。
3. 用 `auth()->user()->resumes()->create($data)`。
4. redirect 回列表。

```php
public function save(): void
{
    // 範例用途：送出 Livewire 表單並建立履歷。
    // 主要輸入：title、summary public properties。
    // 回傳結果 / 副作用：寫入 resumes 資料表。
    $data = $this->validate();

    auth()->user()->resumes()->create($data);

    $this->redirectRoute('resumes.index');
}
```

### 如果結果和預期不同

- `Call to a member function resumes() on null`：使用者未登入。
- Mass assignment 錯誤：Resume model 缺 fillable。

### 負面例子 / 錯誤用法

錯誤做法：`Resume::create($data + ['user_id' => request('user_id')])`。
問題：使用者可冒用別人的 user_id。
修正方向：user_id 由登入者關聯決定。

### 小練習

儲存成功後顯示 flash message。

### Junior 常見誤解

表單送出不只是把資料塞進 DB，還要確保資料屬於正確使用者。

### 一句話總結

Livewire 表單保存資料時，登入者與驗證是兩條底線。

---

## Day 16：Resume Builder 專案發想與架構

### 這篇文章主要在講什麼

開始 resume builder 小專案，先定義功能、資料模型與頁面流程。

### 為什麼需要這個概念

直接開寫容易漏掉資料關係與權限。先畫出功能範圍，後面 CRUD 才不會亂。

### 學完這篇你應該會做到什麼

你應該能列出 resume builder 的 MVP 功能與資料表。

### 核心重點

- MVP：登入、建立履歷、列表、查看、編輯、刪除。
- 一個 user 有多份 resume。
- resume 可以有 skill、experience、education。
- 先做最小可用，再擴充。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「產品設計」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 產品設計 | 定義 Resume Builder MVP | 有 user story、資料表與頁面流程 |

當天接到主線專案的流程：

1. 列出登入、CRUD、發布 GitHub README。
2. 畫出 User 與 Resume 關聯。
3. 決定先做主表，後續再拆 skills/experiences。

### 當天做完後檢查

- MVP 與 later 功能分清楚。
- 每個頁面能對應一個 route。
- 資料模型不過度設計。
### 範例範圍地圖

| 部分 | 資料表 / 頁面 | 負責的事情 |
| --- | --- | --- |
| User | `users` | 登入者 |
| Resume | `resumes` | 履歷主資料 |
| Skill | `resume_skills` | 技能列表 |
| Pages | index/create/show/edit | CRUD 頁面 |

### 完整實作流程、範例與注意事項

1. 寫出 user story。
2. 畫出資料關係。
3. 決定先做 `resumes` 主表。
4. 拆 route：index、create、store、show、edit、update、destroy。

```text
範例 user story：
身為登入使用者，我想建立一份履歷，讓我之後可以更新 GitHub README。

驗收條件：
1. 未登入不能建立履歷。
2. 履歷至少要有 title。
3. 使用者只能看到自己的履歷。
```

### 如果結果和預期不同

- 功能越列越多：先標 MVP 與 later。
- 資料表太複雜：先做主表，關聯表逐步加。

### 負面例子 / 錯誤用法

錯誤做法：還沒定義資料關係就開始做畫面。
問題：後面會一直改 migration 與 controller。
修正方向：先列 MVP 與資料模型。

### 小練習

替 resume builder 寫 5 條 user story。

### Junior 常見誤解

小專案也需要設計，只是設計要輕量。

### 一句話總結

先定義 MVP，CRUD 才有方向。

---

## Day 17：Resume Builder 登入畫面與功能上

### 這篇文章主要在講什麼

建立登入功能的前半段：安裝 starter kit 或建立登入頁面。

### 為什麼需要這個概念

Resume 是私人資料，沒有登入就無法判斷資料屬於誰。

### 學完這篇你應該會做到什麼

你應該能用 Laravel Breeze 建立基本登入、註冊、登出流程。

### 核心重點

- Laravel Breeze 適合入門認證。
- Auth 使用 session 管理登入狀態。
- 密碼要 hash，不可明文保存。
- 登入表單也需要 CSRF。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「認證起點」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 認證起點 | 安裝 Breeze | /login、/register 可用 |

當天接到主線專案的流程：

1. 安裝 Breeze。
2. 跑 migration 與 Vite。
3. 建立測試帳號登入。

### 當天做完後檢查

- users table 存在。
- 密碼不是明文保存。
- 登入頁樣式正常。
### 範例範圍地圖

| 部分 | 指令 / 位置 | 負責的事情 |
| --- | --- | --- |
| Breeze | `composer require laravel/breeze --dev` | 建立 auth scaffold |
| Route | `routes/auth.php` | 登入註冊路由 |
| View | `resources/views/auth` | Auth 畫面 |

### 完整實作流程、範例與注意事項

1. 安裝 Breeze。
2. 執行 `php artisan breeze:install blade`。
3. 執行 `npm install && npm run dev`。
4. 執行 migration。
5. 註冊一個測試帳號。

```powershell
# 範例用途：快速建立 Laravel 基本認證。
# 預期結果：出現 /login、/register、/dashboard。
composer require laravel/breeze --dev
php artisan breeze:install blade
php artisan migrate
npm install
npm run dev
```

### 如果結果和預期不同

- 登入頁樣式壞掉：確認 Vite 有跑。
- users table 不存在：執行 migration。

### 負面例子 / 錯誤用法

錯誤做法：自己用 `md5($password)` 存密碼。
問題：不安全且無法抵抗現代攻擊。
修正方向：使用 Laravel Auth 與 `Hash::make()`。

### 小練習

安裝 Breeze 後，建立一個測試帳號並登入 dashboard。

### Junior 常見誤解

登入不是只檢查帳密，還包含 session、CSRF、hash、redirect 等流程。

### 一句話總結

認證功能能用官方 starter kit 就先用，別從零手刻密碼流程。

---

## Day 18：Resume Builder 登入畫面與功能下

### 這篇文章主要在講什麼

完成登入後保護頁面，讓只有登入者能進入 resume builder。

### 為什麼需要這個概念

登入頁存在不代表資料安全，route 也必須被 middleware 保護。

### 學完這篇你應該會做到什麼

你應該能用 `auth` middleware 保護 CRUD route。

### 核心重點

- `middleware('auth')` 限制登入者。
- 未登入會被導到 login。
- 登入者 ID 應決定資料擁有者。
- 測試未登入行為很重要。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「保護路由」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 保護路由 | 用 auth middleware 保護 resumes | 未登入會被導向 login |

當天接到主線專案的流程：

1. 把 resume routes 包進 auth middleware。
2. 未登入測試 /resumes。
3. 登入後再測試 CRUD 入口。

### 當天做完後檢查

- 未登入不能進入履歷列表。
- 不是只靠 navbar 隱藏連結。
- route cache 不會保留舊設定。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Route group | `routes/web.php` | 保護 resumes |
| Auth | Breeze routes | 登入流程 |

### 完整實作流程、範例與注意事項

1. 把 resume routes 包進 auth middleware。
2. 未登入開 `/resumes` 應導到 `/login`。
3. 登入後再開 `/resumes`。

```php
Route::middleware('auth')->group(function () {
    Route::resource('resumes', ResumeController::class);
});
```

### 如果結果和預期不同

- 未登入仍可進入：route 沒包 middleware 或 route cache 未清。
- 登入後 403：可能是 policy 擋住，不是 auth 問題。

### 負面例子 / 錯誤用法

錯誤做法：只在 Blade 隱藏按鈕，不保護 route。
問題：使用者仍可直接輸入 URL。
修正方向：route 層加 middleware。

### 小練習

寫一個 feature test，確認未登入訪問 `/resumes` 會 redirect login。

### Junior 常見誤解

畫面看不到不代表不能呼叫，權限要在後端擋。

### 一句話總結

保護資料從 route middleware 開始。

---

## Day 19：登出功能與 Navbar

### 這篇文章主要在講什麼

依登入狀態顯示 navbar，並完成登出流程。

### 為什麼需要這個概念

使用者需要清楚知道自己是否登入，以及如何安全結束 session。

### 學完這篇你應該會做到什麼

你應該能在 Blade 用 `@auth` / `@guest` 顯示不同導覽列。

### 核心重點

- 登出通常是 POST，不是 GET。
- Navbar 顯示需依 auth state。
- CSRF 仍然需要。
- 登出後應 redirect 到公開頁或 login。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「登入狀態 UI」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 登入狀態 UI | Navbar 顯示登入/登出 | 登入者看到我的履歷與登出 |

當天接到主線專案的流程：

1. Layout 用 auth/guest 判斷。
2. 登出使用 POST form。
3. 登出後 redirect 到公開頁或 login。

### 當天做完後檢查

- 登出 form 有 CSRF。
- 沒有用 GET 刪 session。
- 登入者名稱顯示正常。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Navbar | layout | 顯示連結 |
| Logout form | layout | POST logout |

### 完整實作流程、範例與注意事項

1. 在 layout 加 `@auth` 與 `@guest`。
2. 登出用 form POST 到 logout route。
3. 加 `@csrf`。

```blade
@auth
    <a href="{{ route('resumes.index') }}">我的履歷</a>
    <form method="POST" action="{{ route('logout') }}">
        @csrf
        <button type="submit">登出</button>
    </form>
@else
    <a href="{{ route('login') }}">登入</a>
@endauth
```

### 如果結果和預期不同

- 登出 419：缺 CSRF。
- route 不存在：確認 Breeze auth routes 有載入。

### 負面例子 / 錯誤用法

錯誤做法：用 `<a href="/logout">登出</a>`。
問題：GET request 不應改變登入狀態。
修正方向：用 POST form。

### 小練習

Navbar 顯示登入者姓名。

### Junior 常見誤解

登出是狀態改變，所以應該用 POST。

### 一句話總結

Navbar 是登入狀態的提示，但真正安全仍靠後端 route。

---

## Day 20：Resume Builder 建立填寫表單

### 這篇文章主要在講什麼

建立履歷 create form，讓使用者輸入 title、summary 等資料。

### 為什麼需要這個概念

CRUD 的 C 是所有資料流的起點，表單設計會影響 validation、資料表與使用者體驗。

### 學完這篇你應該會做到什麼

你應該能建立 create 頁與 store action。

### 核心重點

- create 顯示表單。
- store 驗證並寫入。
- old input 保留資料。
- 使用登入者建立關聯。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「Create」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| Create | 完成 create/store | 使用者可新增自己的履歷 |

當天接到主線專案的流程：

1. create 顯示表單。
2. store 驗證 title/summary。
3. 用 request user resumes create 儲存。

### 當天做完後檢查

- 缺 title 會回表單。
- 資料屬於登入者。
- 成功後回列表。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| View | `resumes/create.blade.php` | 表單 |
| Controller | `store()` | 驗證與保存 |
| Model | `Resume` | 寫入資料 |

### 完整實作流程、範例與注意事項

1. 實作 `create()` 回傳 view。
2. 建立 form。
3. 實作 `store()` validation。
4. 儲存後 redirect index。

```php
public function store(Request $request): RedirectResponse
{
    // 範例用途：建立登入者的一份履歷。
    // 主要輸入：POST title、summary。
    // 回傳結果 / 副作用：寫入 resumes，導回列表。
    $data = $request->validate([
        'title' => ['required', 'string', 'max:120'],
        'summary' => ['nullable', 'string', 'max:1000'],
    ]);

    $request->user()->resumes()->create($data);

    return redirect()->route('resumes.index');
}
```

### 如果結果和預期不同

- `resumes()` 不存在：User model 沒定義 relationship。
- 表單送出 419：缺 `@csrf`。

### 負面例子 / 錯誤用法

錯誤做法：讓使用者在 hidden input 傳 `user_id`。
問題：可被竄改。
修正方向：用 `$request->user()`。

### 小練習

替 title 加唯一性限制：同一個使用者不能有同名履歷。

### Junior 常見誤解

Hidden input 不安全，它只是畫面上看不到。

### 一句話總結

建立資料時，擁有者應由登入狀態決定。

---

## Day 21：顯示建立的 Resume List

### 這篇文章主要在講什麼

建立履歷列表頁，顯示登入者自己的履歷。

### 為什麼需要這個概念

列表頁是使用者管理資料的主要入口，也最容易不小心顯示別人的資料。

### 學完這篇你應該會做到什麼

你應該能查詢目前登入者的 resumes 並分頁顯示。

### 核心重點

- 查詢要限制登入者。
- 大量資料用 pagination。
- 空清單要有 empty state。
- link 用 named route。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「Index」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| Index | 完成履歷列表 | 只顯示登入者自己的履歷 |

當天接到主線專案的流程：

1. index 從登入者 resumes 查。
2. 加上 pagination。
3. 用 empty state 處理無資料。

### 當天做完後檢查

- A 看不到 B 的履歷。
- 大量資料不一次全撈。
- 列表連結使用 named route。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Controller | `index()` | 查詢自己的資料 |
| View | `index.blade.php` | 顯示列表 |

### 完整實作流程、範例與注意事項

1. 在 `index()` 使用 `$request->user()->resumes()`。
2. 加 `latest()` 與 `paginate(10)`。
3. Blade 用 `@forelse` 顯示 empty state。

```php
public function index(Request $request): View
{
    $resumes = $request->user()
        ->resumes()
        ->latest()
        ->paginate(10);

    return view('resumes.index', ['resumes' => $resumes]);
}
```

```blade
@forelse ($resumes as $resume)
    <a href="{{ route('resumes.show', $resume) }}">{{ $resume->title }}</a>
@empty
    <p>目前還沒有履歷。</p>
@endforelse

{{ $resumes->links() }}
```

### 如果結果和預期不同

- 看得到別人的資料：查詢沒有從登入者關聯開始。
- 分頁連結壞掉：確認 view 有載入 pagination 樣式或預設樣式。

### 負面例子 / 錯誤用法

錯誤做法：`Resume::latest()->paginate(10)`。
問題：會顯示所有使用者資料。
修正方向：`$request->user()->resumes()`。

### 小練習

在列表加搜尋 title 的功能。

### Junior 常見誤解

列表查詢不是拿全部資料，而是拿「目前使用者有權看的資料」。

### 一句話總結

列表頁的第一條件永遠是資料範圍。

---

## Day 22：顯示單筆 Resume 頁面

### 這篇文章主要在講什麼

建立 show 頁，顯示單筆履歷內容。

### 為什麼需要這個概念

單筆頁看似簡單，但權限與 404 處理是基本功。

### 學完這篇你應該會做到什麼

你應該能顯示單筆資料，並阻止使用者查看別人的履歷。

### 核心重點

- Route model binding 查資料。
- Policy 或手動檢查擁有者。
- 找不到資料回 404。
- 沒權限回 403。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「Show + Policy」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| Show + Policy | 完成單筆頁與 view 權限 | 不能查看別人的履歷 |

當天接到主線專案的流程：

1. 建立 ResumePolicy。
2. show 前呼叫 authorize。
3. Blade 顯示單筆資料。

### 當天做完後檢查

- 無權限回 403。
- 不存在資料回 404。
- show 頁沒有洩漏其他使用者資料。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Controller | `show()` | 顯示單筆 |
| Policy | `ResumePolicy` | 權限檢查 |

### 完整實作流程、範例與注意事項

1. 建立 `ResumePolicy`。
2. 在 `view()` 檢查 user id。
3. Controller 呼叫 `$this->authorize('view', $resume)`。

```php
public function show(Resume $resume): View
{
    // 範例用途：顯示單筆履歷前先檢查權限。
    // 主要輸入：URL 中的 resume id。
    // 回傳結果：有權限顯示，沒權限 403。
    $this->authorize('view', $resume);

    return view('resumes.show', ['resume' => $resume]);
}
```

```php
public function view(User $user, Resume $resume): bool
{
    return $resume->user_id === $user->id;
}
```

### 如果結果和預期不同

- 403：policy 判斷不通過。
- 404：id 不存在或 route model binding 找不到。

### 負面例子 / 錯誤用法

錯誤做法：show 頁不做權限，靠列表不顯示別人的連結。
問題：使用者可猜 URL。
修正方向：後端每個單筆操作都檢查權限。

### 小練習

替 edit、update、delete 也套用 policy。

### Junior 常見誤解

使用者看不到連結，不代表他不能直接打 URL。

### 一句話總結

單筆頁要同時處理「資料存在」與「使用者有權限」。

---

## Day 23：編輯 Resume 上

### 這篇文章主要在講什麼

建立 edit 頁，顯示現有資料並讓使用者修改。

### 為什麼需要這個概念

編輯表單和建立表單很像，但多了「預填資料」與「權限」。

### 學完這篇你應該會做到什麼

你應該能建立 edit view，讓欄位顯示原本的資料。

### 核心重點

- edit 要先 authorize。
- input value 使用 `old('field', $model->field)`。
- form method 用 POST 加 `@method('PUT')`。
- action 指向 update route。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「Edit form」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| Edit form | 建立 edit 頁與預填資料 | 表單顯示原履歷內容 |

當天接到主線專案的流程：

1. edit 前先 authorize。
2. 表單 action 指向 update。
3. 欄位使用 old value fallback model value。

### 當天做完後檢查

- PUT method spoofing 存在。
- 驗證失敗後保留輸入。
- 不會覆蓋成空白表單。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Controller | `edit()` | 顯示表單 |
| View | `edit.blade.php` | 預填欄位 |

### 完整實作流程、範例與注意事項

1. 實作 `edit(Resume $resume)`。
2. 檢查權限。
3. 回傳 view。
4. 表單預填資料。

```blade
<form method="POST" action="{{ route('resumes.update', $resume) }}">
    @csrf
    @method('PUT')

    <input name="title" value="{{ old('title', $resume->title) }}">
    <textarea name="summary">{{ old('summary', $resume->summary) }}</textarea>

    <button type="submit">更新</button>
</form>
```

### 如果結果和預期不同

- 表單送到 store：action route 寫錯。
- PUT 不生效：缺 `@method('PUT')`。

### 負面例子 / 錯誤用法

錯誤做法：edit 頁直接用空白 create form。
問題：使用者不知道原本資料，也容易覆蓋成空值。
修正方向：使用 `old(..., $resume->...)`。

### 小練習

把 create/edit 共用成 partial form。

### Junior 常見誤解

HTML form 只支援 GET/POST，Laravel 用 method spoofing 模擬 PUT/DELETE。

### 一句話總結

編輯表單的關鍵是權限、預填與正確 HTTP method。

---

## Day 24：編輯 Resume 下

### 這篇文章主要在講什麼

完成 update action，驗證並保存修改。

### 為什麼需要這個概念

更新資料比建立更容易有覆蓋風險，尤其是同時多人操作時。

### 學完這篇你應該會做到什麼

你應該能安全更新一份履歷並回到 show 頁。

### 核心重點

- update 要 authorize。
- validation 規則可和 store 共用。
- 更新後 redirect show。
- 複雜專案要注意 optimistic locking。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「Update」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| Update | 完成 update action | 可修改履歷並回 show |

當天接到主線專案的流程：

1. update 前 authorize。
2. validate 後只更新允許欄位。
3. 成功後 redirect show。

### 當天做完後檢查

- 沒有 request all。
- fillable 包含可更新欄位。
- 更新別人資料會 403。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Controller | `update()` | 驗證與保存 |
| Policy | `update()` | 權限 |

### 完整實作流程、範例與注意事項

1. Controller 收到 PUT request。
2. authorize update。
3. validate。
4. `$resume->update($data)`。
5. redirect show。

```php
public function update(Request $request, Resume $resume): RedirectResponse
{
    $this->authorize('update', $resume);

    $data = $request->validate([
        'title' => ['required', 'string', 'max:120'],
        'summary' => ['nullable', 'string', 'max:1000'],
    ]);

    $resume->update($data);

    return redirect()
        ->route('resumes.show', $resume)
        ->with('status', '履歷已更新');
}
```

### 如果結果和預期不同

- 沒有更新：確認 `$fillable` 包含欄位。
- 403：policy 不允許。

### 負面例子 / 錯誤用法

錯誤做法：`$resume->update($request->all())`。
問題：可被 mass assignment 攻擊。
修正方向：只用 validated data。

### 小練習

把 store/update validation 抽到 `ResumeRequest`。

### Junior 常見誤解

更新資料不是「把 request 全丟進 model」，而是只更新被允許的欄位。

### 一句話總結

Update 的安全核心是 authorize 後 validate，再 update。

---

## Day 25：刪除單筆 Resume 功能

### 這篇文章主要在講什麼

實作 destroy，刪除指定履歷。

### 為什麼需要這個概念

刪除是高風險操作，一定要確認權限與使用者意圖。

### 學完這篇你應該會做到什麼

你應該能用 DELETE request 刪除自己的履歷。

### 核心重點

- 刪除要用 POST form + `@method('DELETE')`。
- destroy 要 authorize。
- 刪除前 UI 應確認。
- 真實產品常用 soft delete。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「Delete」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| Delete | 完成 destroy action | 可刪除自己的履歷 |

當天接到主線專案的流程：

1. 刪除按鈕用 POST form + DELETE method。
2. destroy 前 authorize。
3. 刪除後回列表。

### 當天做完後檢查

- 不是 GET 刪除。
- 有確認提示或安全 UI。
- 誤刪需求可改 soft delete。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| View | show/edit | 刪除按鈕 |
| Controller | `destroy()` | 刪除資料 |

### 完整實作流程、範例與注意事項

1. 在 show 頁建立 delete form。
2. 加 `@csrf` 與 `@method('DELETE')`。
3. Controller authorize。
4. 刪除後 redirect index。

```php
public function destroy(Resume $resume): RedirectResponse
{
    $this->authorize('delete', $resume);

    $resume->delete();

    return redirect()
        ->route('resumes.index')
        ->with('status', '履歷已刪除');
}
```

```blade
<form method="POST" action="{{ route('resumes.destroy', $resume) }}">
    @csrf
    @method('DELETE')
    <button type="submit" onclick="return confirm('確定刪除？')">刪除</button>
</form>
```

### 如果結果和預期不同

- 405 Method Not Allowed：form method 或 `@method` 錯。
- 誤刪資料：考慮 soft delete。

### 負面例子 / 錯誤用法

錯誤做法：用 GET `/resumes/1/delete` 刪除。
問題：搜尋引擎、預載、誤點都可能造成刪除。
修正方向：使用 DELETE method。

### 小練習

替 Resume model 加 SoftDeletes。

### Junior 常見誤解

刪除不是普通連結，它是有副作用的操作。

### 一句話總結

刪除功能要保守設計：權限、確認、正確 method。

---

## Day 26：將 Resume 更新至 GitHub README 上

### 這篇文章主要在講什麼

開始把履歷內容輸出成 Markdown，準備更新 GitHub README。

### 為什麼需要這個概念

外部整合前要先把內部資料轉成穩定格式，不能把 view HTML 直接拿去發 API。

### 學完這篇你應該會做到什麼

你應該能把 Resume model 轉成 Markdown 字串。

### 核心重點

- 先做 Markdown formatter。
- formatter 不應呼叫 GitHub API。
- 測試輸出格式。
- 注意 Markdown escaping。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「Markdown 輸出」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| Markdown 輸出 | 建立 ResumeMarkdownFormatter | 履歷可轉成 Markdown |

當天接到主線專案的流程：

1. 建立 formatter class。
2. 把 title/summary 轉成 Markdown。
3. 先在本機預覽輸出。

### 當天做完後檢查

- formatter 不呼叫 GitHub API。
- Markdown 格式穩定。
- 使用者輸入不破壞格式。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Formatter | `ResumeMarkdownFormatter` | 把 resume 轉 markdown |
| Controller | publish action | 呼叫 formatter |

### 完整實作流程、範例與注意事項

1. 建立 formatter class。
2. 傳入 Resume。
3. 回傳 markdown。
4. 先在頁面預覽或 log 檢查。

```php
<?php

class ResumeMarkdownFormatter
{
    public function format(Resume $resume): string
    {
        // 範例用途：把履歷資料轉成 GitHub README 可用 Markdown。
        // 主要輸入：Resume model。
        // 回傳結果：Markdown 字串，不直接呼叫外部 API。
        return <<<MD
# {$resume->title}

{$resume->summary}
MD;
    }
}
```

### 如果結果和預期不同

- Markdown 斷行怪：確認換行與空行。
- 使用者輸入破壞格式：需要 escape 或限制欄位。

### 負面例子 / 錯誤用法

錯誤做法：controller 裡一邊組 markdown 一邊呼叫 API。
問題：格式與外部整合耦合，難測試。
修正方向：formatter 與 API service 分開。

### 小練習

為 formatter 寫測試，確認 title 會變成 H1。

### Junior 常見誤解

輸出格式也是一層邏輯，不是隨便字串相加。

### 一句話總結

先把履歷穩定轉成 Markdown，再談 GitHub API。

---

## Day 27：OAuth Token and Scope

### 這篇文章主要在講什麼

理解 GitHub token 與 scope，知道更新 README 需要什麼權限。

### 為什麼需要這個概念

Token 權限太大會造成安全風險，權限太小會導致 API 失敗。

### 學完這篇你應該會做到什麼

你應該能用 fine-grained token，只開指定 repo 的 Contents read/write。

### 核心重點

- Token 不可 commit。
- fine-grained token 優於過大 classic token。
- scope 只開需要的權限。
- token 放 `.env` 或 secret manager。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「GitHub Token」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| GitHub Token | 設定最小權限 token | GitHub token 由 config 讀取 |

當天接到主線專案的流程：

1. 建 fine-grained token。
2. 只開指定 repo Contents read/write。
3. 寫入 .env，不進 git。

### 當天做完後檢查

- .env.example 只有 placeholder。
- token 沒有出現在程式碼。
- 401/403 能分辨原因。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| `.env` | `GITHUB_TOKEN=...` | 本機 token |
| Config | `services.github.token` | 統一讀取 |
| Service | API call | 使用 token |

### 完整實作流程、範例與注意事項

1. 建立 fine-grained token。
2. Repository access 選指定 repo。
3. Permissions 開 Contents read/write。
4. 放進 `.env`。
5. `config/services.php` 讀取。

```php
'github' => [
    // 範例用途：集中管理 GitHub token。
    // 注意：env 值不要 commit，正式環境改用平台 secret。
    'token' => env('GITHUB_TOKEN'),
],
```

### 如果結果和預期不同

- 401：token 錯誤。
- 403：scope 不足或 repo 沒授權。
- token 外洩：立刻 revoke，重新產生。

### 負面例子 / 錯誤用法

錯誤做法：把 token 寫死在 class。
問題：進 git history 後很難完全清除。
修正方向：放環境變數與 secret manager。

### 小練習

建立 `.env.example` 的 `GITHUB_TOKEN=` placeholder，但不要填真值。

### Junior 常見誤解

Token 就是另一種密碼，不能因為它是 API 用的就隨便放。

### 一句話總結

GitHub token 的原則是最小權限與不進版控。

---

## Day 28：發送 PUT Request 更新 GitHub README

### 這篇文章主要在講什麼

使用 GitHub Contents API，以 PUT request 更新 README。

### 為什麼需要這個概念

GitHub 更新檔案不是只送新內容，還需要目前檔案的 SHA 來避免覆蓋別人的修改。

### 學完這篇你應該會做到什麼

你應該能先 GET README 取得 SHA，再 PUT 更新內容。

### 核心重點

- GET `/contents/README.md` 取得 `sha`。
- PUT body 要有 `message`、`content`、`sha`。
- `content` 要 Base64 encode。
- 409 conflict 要重新讀最新 SHA。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「GitHub PUT」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| GitHub PUT | 完成 GitHubReadmeService | 可用 SHA 更新 README |

當天接到主線專案的流程：

1. GET README 取得 sha。
2. formatter 產生 markdown。
3. PUT Base64 content 與 sha。

### 當天做完後檢查

- 409 conflict 會重新讀 sha。
- PUT 成功會建立 GitHub commit。
- service 和 controller 分工清楚。
### 範例範圍地圖

| 部分 | 位置 | 負責的事情 |
| --- | --- | --- |
| Service | `GitHubReadmeService` | GET/PUT GitHub API |
| Formatter | `ResumeMarkdownFormatter` | 產生 content |

### 完整實作流程、範例與注意事項

1. GET 目前 README metadata。
2. 取出 `sha`。
3. 產生新 markdown。
4. Base64 encode。
5. PUT 更新。

```php
public function updateReadme(string $owner, string $repo, string $markdown, string $sha): array
{
    // 範例用途：更新 GitHub README。
    // 主要輸入：owner/repo、markdown、目前 README sha。
    // 回傳結果 / 副作用：GitHub repo 產生一個 commit。
    $response = Http::withToken(config('services.github.token'))
        ->acceptJson()
        ->put("https://api.github.com/repos/{$owner}/{$repo}/contents/README.md", [
            'message' => 'Update resume README',
            'content' => base64_encode($markdown),
            'sha' => $sha,
        ]);

    $response->throw();

    return $response->json();
}
```

### 如果結果和預期不同

- 409：SHA 過期，重新 GET。
- 422：Base64 或 body 欄位錯誤。
- 403：token 權限不足。

### 負面例子 / 錯誤用法

錯誤做法：省略 `sha` 直接 PUT。
問題：GitHub 會拒絕或無法判斷你更新的是哪個版本。
修正方向：先 GET 最新檔案 metadata。

### 小練習

實作 `getReadmeSha($owner, $repo)`。

### Junior 常見誤解

PUT 更新檔案不是覆蓋本機檔案，而是在遠端 repo 建立一次 commit。

### 一句話總結

GitHub Contents API 的更新流程是先讀 SHA，再 PUT 新內容。

---

## Day 29：程式碼優化：使用者登入狀態檢查

### 這篇文章主要在講什麼

整理重複的登入與權限檢查，讓程式更乾淨。

### 為什麼需要這個概念

功能能跑不代表好維護。重複判斷散落各處，之後很容易漏改。

### 學完這篇你應該會做到什麼

你應該能把登入檢查交給 middleware，權限交給 policy。

### 核心重點

- 登入：middleware。
- 擁有者權限：policy。
- 外部 API：service。
- 格式轉換：formatter。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「重構」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 重構 | 把登入/權限/API/格式化歸位 | Controller 變薄，流程更好測 |

當天接到主線專案的流程：

1. auth 交給 middleware。
2. ownership 交給 policy。
3. GitHub API 交給 service。

### 當天做完後檢查

- 沒有到處手寫 auth check。
- 單筆操作都有 policy。
- 重複 validation 可抽 Form Request。
### 範例範圍地圖

| 問題 | 放置位置 |
| --- | --- |
| 未登入不能進入 | route middleware |
| 不能改別人履歷 | policy |
| GitHub API | service |
| Markdown 輸出 | formatter |

### 完整實作流程、範例與注意事項

1. Route 加 `auth`。
2. Controller 每個單筆操作呼叫 authorize。
3. API call 移到 service。
4. 重複 validation 移到 Form Request。

```php
Route::middleware('auth')->group(function () {
    Route::resource('resumes', ResumeController::class);
    Route::post('/resumes/{resume}/publish', PublishResumeController::class)
        ->name('resumes.publish');
});
```

### 如果結果和預期不同

- 重構後 route 壞掉：先跑 `php artisan route:list`。
- 權限錯誤：補 feature test 比人工點更可靠。

### 負面例子 / 錯誤用法

錯誤做法：每個 method 開頭都手寫 `if (!auth()->check())`。
問題：重複且容易漏。
修正方向：middleware。

### 小練習

替 `ResumePolicy` 的 view/update/delete 各寫一個測試。

### Junior 常見誤解

重構不是把程式變漂亮，而是降低改需求時出錯機率。

### 一句話總結

登入、權限、API、格式化各自歸位，程式才會長得穩。

---

## Day 30：完結與下一步

### 這篇文章主要在講什麼

回顧 30 天學到的 Laravel 基礎與 resume builder 專案。

### 為什麼需要這個概念

完賽不是結束，而是把學過的東西變成下一個可維護專案的基礎。

### 學完這篇你應該會做到什麼

你應該能說出 Laravel 專案從 request 到 response 的主要流程，並知道下一步該補哪些能力。

### 核心重點

- CRUD 是後端基本功。
- Auth 與 authorization 必須分清楚。
- 外部 API 要抽 service。
- 測試是下一階段必修。


### 主線專案銜接

這一天要接回 resume-builder 主線，而不是只停在單一語法。完成「收尾」後，讀者應該能在同一個專案裡看到明確進度。

| 主線部分 | 這一天要補上的東西 | 完成後的可見結果 |
| --- | --- | --- |
| 收尾 | 補測試、README、部署檢查 | 專案可被重新啟動與驗證 |

當天接到主線專案的流程：

1. 跑 migrate fresh 與 test。
2. 補 README 啟動方式。
3. 檢查 GitHub token 與權限。

### 當天做完後檢查

- php artisan test 通過或知道失敗點。
- README 能讓別人啟動。
- 主線功能手動走完一次。
### 範例範圍地圖

| 下一步 | 練習方向 |
| --- | --- |
| Testing | Feature test、Pest |
| Queue | GitHub README 更新改成背景工作 |
| API | Sanctum 建 API token |
| Deployment | Laravel Forge、Docker、GitHub Actions |

### 完整實作流程、範例與注意事項

1. 跑完整測試或手動 checklist。
2. 確認未登入不能進 resume。
3. 確認 A 使用者不能操作 B 使用者資料。
4. 確認 GitHub token 沒 commit。
5. 寫 README 記錄專案啟動方式。

```powershell
# 範例用途：完賽後基本檢查。
php artisan test
php artisan route:list
php artisan migrate:fresh --seed
```

### 如果結果和預期不同

- 測試失敗：先看第一個失敗，不要一次改全部。
- migrate fresh 失敗：migration 或 seeder 有問題。

### 負面例子 / 錯誤用法

錯誤做法：功能做完就不整理、不測試、不寫啟動說明。
問題：一週後自己也很難接回來。
修正方向：補 README、測試與 checklist。

### 小練習

把 GitHub README 更新改成 queue job，並加一個失敗重試紀錄。

### Junior 常見誤解

完成畫面不等於完成專案；能維護、能部署、能排錯才算更完整。

### 一句話總結

30 天學 Laravel 的成果，是你開始能用框架思考一個完整 Web 功能。
