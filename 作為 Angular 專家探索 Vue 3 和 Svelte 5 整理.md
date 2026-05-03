<h1 style="font-size: 2.2em;">作為 Angular 專家探索 Vue 3 和 Svelte 5 整理</h1>

## 來源

- 系列頁：<https://ithelp.ithome.com.tw/users/20168314/ironman/8200>
- 作者：Connie（connieleung）
- 主題：以 Angular 開發者視角，透過 Vue 3、Svelte 5 / SvelteKit 與 Angular 19/20 重作相同小專案，比較三個框架的元件、狀態、資料流、部署與非同步資料處理。

## 這份筆記怎麼讀

這個系列不是單純教 Vue 或 Svelte，而是用「同一個功能在三個框架怎麼做」來建立前端框架的共同心智模型。讀者如果已經懂 Angular，可以把 Vue 的 `ref`、`computed`、`defineProps`、`defineEmits`，Svelte 的 `$state`、`$derived`、`$props`，對照到 Angular 的 `signal`、`computed`、`input`、`output`、`model`。

建議閱讀順序：

1. Day 0 到 Day 11：購物車專案，理解模板、清單、事件、條件、樣式、響應式與部署。
2. Day 12 到 Day 17：Coffee Plan 專案，理解元件拆分、props、父子溝通、自訂事件與內容投影。
3. Day 18 到 Day 21：GitHub Card 專案，理解 API 擷取、元件組合、Tailwind / DaisyUI 與 GitHub Pages 部署。
4. Day 22 到 Day 26：Alert 元件，理解動態元件、雙向綁定、狀態封裝與重構。
5. Day 27 到 Day 30：Blog 專案與回顧，理解路由、資料載入、關聯資料、loading / error 狀態。

## 整體地圖

| 範圍 | 小專案 | 主要學習 |
| --- | --- | --- |
| Day 0-11 | Shopping Cart | 專案建立、模板、事件、清單、條件渲染、樣式綁定、衍生狀態、部署 |
| Day 12-17 | Coffee Plan | 元件拆分、props、父子元件、emit / callback / output、slot / snippet / ng-content |
| Day 18-21 | GitHub Card | API token、資料擷取、元件組合、Tailwind / DaisyUI、GitHub Pages |
| Day 22-26 | Alert Component | Alert list、動態圖示、雙向綁定、重開已關閉通知、抽取狀態管理 |
| Day 27-30 | Blog | 路由、文章與作者資料、watch / loader、loading / error、系列回顧 |

---

## Day 0：為什麼參加這個挑戰

### 這篇文章主要在講什麼

作者以 Angular GDE 的身份出發，承認自己長期熟悉 Angular，但對 Vue 3、Svelte 5 這些現代框架缺少實戰經驗。這個挑戰的目標，是用 Vue School 課程為基礎，把同一批練習用 Vue 3、Angular 與 Svelte 5 重做，建立跨框架比較能力。

### 為什麼需要這個概念

前端框架會變，但元件化、狀態、輸入輸出、事件、資料載入這些核心概念不會消失。只學某一個框架 API，容易把框架語法當成全部；跨框架練習能逼自己看懂概念本身。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 0：為什麼參加這個挑戰 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 已熟悉框架 | Angular | 作者原本的主要技能與比較基準 |
| 新學習框架 | Vue 3、Svelte 5 | 用來重做相同範例，觀察差異 |
| 共同核心 | Reactivity、Input / Output、Component Architecture | 三個框架都需要掌握的底層概念 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 0：為什麼參加這個挑戰 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 先選一個很小的功能，例如顯示標題。
2. 分別用 Vue、Svelte、Angular 寫出同一件事。
3. 比較「狀態宣告」、「模板讀值」、「更新畫面」三個動作。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
// Vue 3：ref 是響應式狀態
const title = ref('Shopping List App')

// Svelte 5：$state 是可更新狀態
let title = $state('Shopping List App')

// Angular：signal 是可追蹤狀態
title = signal('Shopping List App')
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 團隊要評估新框架時，用同一個小功能做 proof of concept。
- Angular 開發者接手 Vue / Svelte 專案時，用熟悉概念對照陌生語法。
- 技術分享時，用「共同問題，不同框架解法」降低學習門檻。

### 不適合使用的情境

- 只為了追新而重寫成熟產品。
- 沒有業務需求，卻把同一產品維護成三份框架版本。

### 負面例子 / 錯誤用法

錯誤做法：看到 Vue 的 `ref` 或 Svelte 的 `$state` 就直接套 Angular 思維，假設所有框架都有相同生命週期與變更偵測。

問題：語法看似相近，但更新時機、編譯模型、資料流方向可能不同，會造成難追的 UI 狀態錯誤。

修正方向：先比較「輸入、狀態、事件、衍生狀態、渲染」這些概念，再學語法。

### Junior 常見誤解

- 「會 Angular 就等於會所有前端框架」：不對，概念可轉移，API 與慣用法仍要重新學。
- 「框架差異只在語法」：不對，編譯、資料流與最佳實務也會影響設計。

### 一句話總結

跨框架學習的重點不是收集語法，而是看見不同框架如何解決同一類前端問題。

---

## Day 1：建立專案、安裝相依套件與全域 CSS

### 這篇文章主要在講什麼

這篇建立三個專案：Vue 3、SvelteKit 與 Angular，並安裝基礎套件、啟動 dev server、放入全域 CSS，讓後續購物車練習有一致起點。

### 為什麼需要這個概念

專案初始化不是雜事。相依套件、TypeScript、ESLint、Prettier、CSS 載入位置，會決定後續開發體驗與團隊一致性。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 1：建立專案、安裝相依套件與全域 CSS 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 專案建立器 | `npm create vue@latest`、`npx sv create`、Angular CLI | 產生可執行專案骨架 |
| 開發伺服器 | `npm run dev` / Angular serve | 本機預覽與熱更新 |
| 全域樣式 | global CSS | 提供跨元件共用視覺基礎 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 1：建立專案、安裝相依套件與全域 CSS 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 建立三個專案並啟用 TypeScript。
2. 安裝相依套件。
3. 啟動本機伺服器確認可編譯。
4. 將共用 CSS 放進各框架慣用的全域入口。

```bash
# 範例用途：示範「實作流程與程式碼例子」中可直接在終端機執行的 Shell 指令。
# 參數說明：命令中的 URL、檔名、路徑、選項或環境名稱請替換成你的實際目標。
# 回傳結果 / 副作用：通常會輸出結果、讀寫檔案、下載資料，或改變目前 shell / 系統狀態。
npm create vue@latest fundamental-vue3
npx sv create fundamental-svelte
ng new fundamental-angular
```

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
// Vue main.ts：入口匯入全域樣式
import './assets/main.css'
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 新專案開工前建立一致的 formatter、lint、TypeScript 設定。
- 教學、POC、框架比較時建立乾淨 baseline。

### 不適合使用的情境

- 已有企業模板或 monorepo 規範時，不應每次用官方 starter 自行分裂設定。

### 負面例子 / 錯誤用法

錯誤做法：三個專案使用不同 CSS reset、不同 TypeScript 嚴格度，然後比較開發體驗。

問題：比較結果會混入工具差異，不是真正的框架差異。

修正方向：先固定基礎條件，再比較元件與狀態寫法。

### Junior 常見誤解

- 「能跑就好」：能跑只是第一步，團隊還需要一致的建置、格式化與路徑結構。

### 一句話總結

好的專案初始化，是後續框架比較與功能開發的乾淨起跑線。

---

## Day 2：建立 ShoppingCart 元件

### 這篇文章主要在講什麼

這篇把畫面抽成 `ShoppingCart` 元件，並在 Vue、Svelte、Angular 中各自掛到根元件。

### 為什麼需要這個概念

元件是前端應用的基本組織單位。即使一開始只有一行文字，也應該練習「建立元件、匯入元件、在模板使用元件」的完整路徑。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 2：建立 ShoppingCart 元件 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 子元件 | `ShoppingCart` | 封裝購物車 UI |
| 根元件 | `App.vue`、`+page.svelte`、Angular App | 掛載子元件 |
| 模板 | template / markup | 顯示元件內容 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 2：建立 ShoppingCart 元件 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 建立 `ShoppingCart` 元件。
2. 在根元件匯入它。
3. 在模板放入 `<ShoppingCart />`。
4. 確認畫面顯示 Shopping Cart。

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<!-- Vue：ShoppingCart.vue -->
<script setup lang="ts"></script>

<template>
  <p>Shopping Cart</p>
</template>
```

```svelte
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<!-- Svelte：shopping-cart.svelte -->
<p>Shopping Cart</p>
```

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
// Angular：shopping-cart.component.ts
@Component({
  selector: 'app-shopping-cart',
  template: '<p>Shopping Cart</p>',
})
export class ShoppingCartComponent {}
```


### 完整範圍補強：同一個 Shopping Cart 功能怎麼跨框架看

這份筆記原本有很多單日元件範例，但前端框架學習不能只看單一 component。正常實作會有「入口檔、父元件、子元件、狀態、事件、樣式、驗證方式」一起配合。下面用購物車的最小功能建立完整範圍，後面 Day 1-11 都可以回到這個地圖對照。

#### 範例範圍地圖

| 部分 | Vue 3 | Svelte 5 | Angular |
| --- | --- | --- | --- |
| 專案入口 | `src/main.ts` | `src/main.ts` 或 SvelteKit route | `src/main.ts`、`app.config.ts` |
| 根元件 | `src/App.vue` | `src/App.svelte` | `app.component.ts` |
| 子元件 | `src/components/ShoppingCart.vue` | `src/lib/ShoppingCart.svelte` | `shopping-cart.component.ts` |
| 狀態 | `ref` / `computed` | `$state` / `$derived` | `signal` / `computed` |
| 事件 | `@click`、`emit` | `onclick`、callback prop | `(click)`、`output` |
| 樣式 | component scoped style 或 CSS | component style | component styles |
| 驗證 | 瀏覽器畫面、console、測試資料 | 瀏覽器畫面、console、測試資料 | 瀏覽器畫面、Angular dev server |

#### 最小資料模型

三個框架都先用同一種資料形狀，這樣比較才公平：

```ts
type CartItem = {
  id: number
  name: string
  price: number
  quantity: number
}
```

#### Vue 3：父元件持有狀態，子元件呈現與觸發事件

`src/App.vue`：

```vue
<script setup lang="ts">
import { computed, ref } from 'vue'
import ShoppingCart from './components/ShoppingCart.vue'

type CartItem = {
  id: number
  name: string
  price: number
  quantity: number
}

// 範例用途：父元件保存購物車狀態。
// 副作用：increaseQuantity 會改變 items，畫面會因 ref 自動更新。
const items = ref<CartItem[]>([
  { id: 1, name: 'Coffee', price: 120, quantity: 1 },
  { id: 2, name: 'Cake', price: 90, quantity: 2 }
])

const total = computed(() =>
  items.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
)

function increaseQuantity(id: number) {
  const item = items.value.find(x => x.id === id)
  if (item) item.quantity++
}
</script>

<template>
  <ShoppingCart
    :items="items"
    :total="total"
    @increase="increaseQuantity"
  />
</template>
```

`src/components/ShoppingCart.vue`：

```vue
<script setup lang="ts">
type CartItem = {
  id: number
  name: string
  price: number
  quantity: number
}

defineProps<{
  items: CartItem[]
  total: number
}>()

const emit = defineEmits<{
  increase: [id: number]
}>()
</script>

<template>
  <section>
    <h1>Shopping Cart</h1>
    <article v-for="item in items" :key="item.id">
      <span>{{ item.name }} x {{ item.quantity }}</span>
      <button @click="emit('increase', item.id)">+</button>
    </article>
    <strong>Total: {{ total }}</strong>
  </section>
</template>
```

#### Svelte 5：父元件用 `$state`，子元件用 callback prop

`src/App.svelte`：

```svelte
<script lang="ts">
  import ShoppingCart from './lib/ShoppingCart.svelte'

  type CartItem = {
    id: number
    name: string
    price: number
    quantity: number
  }

  let items = $state<CartItem[]>([
    { id: 1, name: 'Coffee', price: 120, quantity: 1 },
    { id: 2, name: 'Cake', price: 90, quantity: 2 }
  ])

  let total = $derived(
    items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  )

  function increaseQuantity(id: number) {
    const item = items.find(x => x.id === id)
    if (item) item.quantity += 1
  }
</script>

<ShoppingCart {items} {total} onIncrease={increaseQuantity} />
```

`src/lib/ShoppingCart.svelte`：

```svelte
<script lang="ts">
  type CartItem = {
    id: number
    name: string
    price: number
    quantity: number
  }

  let {
    items,
    total,
    onIncrease
  }: {
    items: CartItem[]
    total: number
    onIncrease: (id: number) => void
  } = $props()
</script>

<section>
  <h1>Shopping Cart</h1>
  {#each items as item (item.id)}
    <article>
      <span>{item.name} x {item.quantity}</span>
      <button onclick={() => onIncrease(item.id)}>+</button>
    </article>
  {/each}
  <strong>Total: {total}</strong>
</section>
```

#### Angular：父元件用 signal，子元件用 input / output

`shopping-cart.component.ts`：

```ts
import { Component, input, output } from '@angular/core'

type CartItem = {
  id: number
  name: string
  price: number
  quantity: number
}

@Component({
  selector: 'app-shopping-cart',
  standalone: true,
  template: `
    <section>
      <h1>Shopping Cart</h1>
      @for (item of items(); track item.id) {
        <article>
          <span>{{ item.name }} x {{ item.quantity }}</span>
          <button (click)="increase.emit(item.id)">+</button>
        </article>
      }
      <strong>Total: {{ total() }}</strong>
    </section>
  `
})
export class ShoppingCartComponent {
  items = input.required<CartItem[]>()
  total = input.required<number>()
  increase = output<number>()
}
```

`app.component.ts`：

```ts
import { Component, computed, signal } from '@angular/core'
import { ShoppingCartComponent } from './shopping-cart.component'

type CartItem = {
  id: number
  name: string
  price: number
  quantity: number
}

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [ShoppingCartComponent],
  template: `
    <app-shopping-cart
      [items]="items()"
      [total]="total()"
      (increase)="increaseQuantity($event)"
    />
  `
})
export class AppComponent {
  items = signal<CartItem[]>([
    { id: 1, name: 'Coffee', price: 120, quantity: 1 },
    { id: 2, name: 'Cake', price: 90, quantity: 2 }
  ])

  total = computed(() =>
    this.items().reduce((sum, item) => sum + item.price * item.quantity, 0)
  )

  increaseQuantity(id: number) {
    this.items.update(items =>
      items.map(item =>
        item.id === id ? { ...item, quantity: item.quantity + 1 } : item
      )
    )
  }
}
```

#### 端到端流程

1. 父元件建立 `items` 狀態與 `total` 衍生狀態。
2. 父元件把 `items` 和 `total` 傳給 `ShoppingCart` 子元件。
3. 子元件顯示清單與總金額。
4. 使用者按 `+`。
5. 子元件透過 emit / callback / output 通知父元件。
6. 父元件修改狀態，框架重新渲染畫面。

#### 做完後檢查

- 畫面顯示 `Coffee`、`Cake` 與總金額。
- 按下 `+` 後，對應商品數量增加。
- 總金額會跟著增加。
- 若畫面不更新，先檢查狀態是否用該框架的響應式 API 建立，而不是普通變數。

#### 後續 Day 怎麼沿用這個範例

- Day 3 的模板表達式，可以直接拿 `total` 與 `item.quantity` 觀察畫面如何讀取狀態。
- Day 4 的清單渲染，可以回到 `items` 與 `v-for` / `{#each}` / `@for` 對照。
- Day 5 到 Day 6 的輸入與事件，可以回到 `increaseQuantity` 與 emit / callback / output。
- Day 8 到 Day 10 的樣式與衍生狀態，可以回到 `total`、按鈕狀態與購物車顯示結果。

### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 將頁面中的功能區塊拆成可測、可替換的元件。
- 讓根頁面只負責組合，不塞入所有 UI 細節。

### 不適合使用的情境

- 對只有一次性、極小且沒有狀態的 HTML 過度拆分，會讓檔案數比問題本身更複雜。

### 負面例子 / 錯誤用法

錯誤做法：把整個 App 的購物車、表單、清單、樣式都塞在根元件。

問題：功能一多，根元件會變成難測、難讀、難重構的巨大檔案。

修正方向：從有明確責任的 UI 區塊開始拆，例如 `ShoppingCart`。

### Junior 常見誤解

- 「元件一定要很複雜才值得建立」：不對，元件的價值在責任邊界，不在程式碼行數。

### 一句話總結

建立第一個元件，是把畫面從一大坨 HTML 變成可組合功能單位的開始。

---

## Day 3：在模板中使用表達式

### 這篇文章主要在講什麼

這篇示範如何在元件中宣告一個 `header` 狀態，並在模板輸出它。Vue 使用 `ref` 與 `{{ }}`，Svelte 使用 `$state` 與 `{ }`，Angular 使用 `signal` 與 `{{ header() }}`。

### 為什麼需要這個概念

模板不是靜態 HTML，它是 UI 與狀態之間的投影。學會把資料顯示到模板，是所有互動功能的第一步。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 3：在模板中使用表達式 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 狀態 | `header` | 儲存要顯示的標題 |
| 模板表達式 | `{{ header }}`、`{header}`、`{{ header() }}` | 把狀態渲染到畫面 |
| 響應式容器 | `ref`、`$state`、`signal` | 讓狀態變動時 UI 可更新 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 3：在模板中使用表達式 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 在元件 script / class 中宣告 `header`。
2. 在模板顯示 `header`。
3. 修改初始值，確認畫面跟著變。

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<script setup lang="ts">
import { ref } from 'vue'

const header = ref('Shopping List App - Vue 3')
</script>

<template>
  <h1>{{ header }}</h1>
</template>
```

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
// Angular
header = signal('Shopping List App - Angular')
```

```html
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<h1>{{ header() }}</h1>
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 顯示頁面標題、使用者名稱、統計數字。
- 將 API 回傳資料放入狀態後顯示在 UI。

### 不適合使用的情境

- 不要把複雜商業邏輯直接寫在模板表達式中。

### 負面例子 / 錯誤用法

錯誤做法：

```html
<!-- 範例用途：示範「負面例子 / 錯誤用法」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<!-- 模板裡塞太多邏輯 -->
<h1>{{ items.filter(x => !x.purchased).map(x => x.label).join(', ') }}</h1>
```

問題：模板難讀，且每次 render 都可能重算。

修正方向：把複雜邏輯抽成 computed / derived / computed signal。

### Junior 常見誤解

- 「模板可以寫 JavaScript，所以邏輯都放模板」：模板應該偏呈現，複雜邏輯放回元件或 service。

### 一句話總結

模板表達式讓狀態可見，但不要讓模板承擔太多邏輯。

---

## Day 4：清單渲染

### 這篇文章主要在講什麼

這篇建立 `Item` 型別與 `items` 清單，並用 Vue 的 `v-for`、Svelte 的 `{#each}`、Angular 的 `@for` 渲染購物車項目。

### 為什麼需要這個概念

大多數前端畫面都在顯示資料集合：商品、訂單、通知、文章、表格列。清單渲染的關鍵是「資料來源、迭代語法、穩定 key」。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 4：清單渲染 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 資料型別 | `Item` | 定義購物車項目的欄位 |
| 清單狀態 | `items` | 保存項目集合 |
| 迭代指令 | `v-for`、`{#each}`、`@for` | 根據資料產生多個 UI 節點 |
| key / track | `item.id` | 幫助框架穩定追蹤每列 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 4：清單渲染 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 定義 `Item` 型別。
2. 建立 `items` 狀態。
3. 用框架的清單語法輸出。
4. 每列使用穩定 ID 作為 key。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
type Item = {
  id: number
  label: string
  purchased: boolean
  highPriority: boolean
}
```

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<li v-for="item in items" :key="item.id">
  {{ item.id }} - {{ item.label }}
</li>
```

```svelte
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
{#each items as item (item.id)}
  <li>{item.id} - {item.label}</li>
{/each}
```

```html
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
@for (item of items(); track item.id) {
  <li>{{ item.id }} - {{ item.label }}</li>
}
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 商品列表、通知列表、留言列表、資料表。
- 使用者可新增、刪除、排序的 UI 集合。

### 不適合使用的情境

- 資料量非常大時，不應直接渲染全部列，應考慮分頁或 virtual scrolling。

### 負面例子 / 錯誤用法

錯誤做法：用陣列 index 當 key。

問題：刪除或排序後，框架可能重用錯誤 DOM，造成表單值、focus 或動畫狀態錯亂。

修正方向：使用資料本身穩定 ID，例如 `item.id`。

### Junior 常見誤解

- 「key 只是為了消除警告」：key 其實是 UI 狀態穩定性的核心。

### 一句話總結

清單渲染不只是迴圈，還要讓框架知道每一列的身份。

---

## Day 5：用戶輸入處理

### 這篇文章主要在講什麼

這篇示範文字輸入與 checkbox 如何綁定到狀態：Vue 用 `v-model`，Svelte 用 `bind:value` / `bind:checked`，Angular 用表單綁定。

### 為什麼需要這個概念

表單是使用者把資料送進系統的主要入口。輸入欄位如果沒有正確綁定，畫面顯示與元件狀態很容易不同步。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 5：用戶輸入處理 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 文字輸入狀態 | `newItem` | 暫存使用者輸入的新項目名稱 |
| 布林狀態 | `newItemHighPriority` | 暫存是否高優先 |
| 雙向綁定 | `v-model`、`bind:value`、form binding | 讓 UI 與狀態同步 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 5：用戶輸入處理 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 建立輸入狀態。
2. 將 input value 綁到狀態。
3. 將 checkbox checked 綁到布林狀態。
4. 暫時把狀態印在畫面上驗證。

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<script setup lang="ts">
import { ref } from 'vue'

const newItem = ref('')
const newItemHighPriority = ref(false)
</script>

<template>
  <input v-model.trim="newItem" name="newItem" />
  <label>
    <input type="checkbox" v-model="newItemHighPriority" />
    High Priority
  </label>
</template>
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 新增待辦、購物車項目、搜尋條件、篩選器。
- 後台表單中的啟用 / 停用、優先級、分類選項。

### 不適合使用的情境

- 高度複雜表單應引入表單驗證策略，而不是只靠簡單雙向綁定。

### 負面例子 / 錯誤用法

錯誤做法：輸入欄位綁定到字串，但 checkbox 綁到字串 `"false"`。

問題：`"false"` 在 JavaScript 中是真值，條件判斷會出錯。

修正方向：checkbox 狀態應該保持 boolean。

### Junior 常見誤解

- 「畫面有顯示輸入值就代表資料正確」：還要確認型別、trim、空字串與驗證規則。

### 一句話總結

輸入處理的核心，是讓使用者輸入與元件狀態保持可信同步。

---

## Day 6：用戶事件處理

### 這篇文章主要在講什麼

這篇加入表單 submit 與刪除按鈕 click。表單送出時新增項目，刪除按鈕點擊時從清單移除項目。

### 為什麼需要這個概念

事件是使用者行為進入應用程式的入口。沒有事件處理，畫面只能顯示資料，不能改變資料。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 6：用戶事件處理 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 新增事件 | submit | 將輸入資料轉成新項目 |
| 刪除事件 | click | 依 ID 移除項目 |
| 事件處理函式 | `saveItem`、`deleteItem` | 封裝狀態更新 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 6：用戶事件處理 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 使用表單收集輸入。
2. 監聽 submit 並阻止預設刷新。
3. 將新項目 push 到 `items`。
4. 清空輸入狀態。
5. 刪除時用 ID 過濾清單。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
const saveItem = () => {
  items.value.push({
    id: Date.now(),
    label: newItem.value,
    highPriority: newItemHighPriority.value,
    purchased: false,
  })
  newItem.value = ''
  newItemHighPriority.value = false
}

const deleteItem = (id: number) => {
  items.value = items.value.filter((item) => item.id !== id)
}
```

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<form @submit.prevent="saveItem">
  <input v-model.trim="newItem" />
  <button type="submit">Save</button>
</form>
```


### 主線落地：事件要接回 Shopping Cart 狀態

範例規則：這段範例必須看成當天 Day 的完整範圍，不是孤立程式碼；先確認前置檔案或前一 Day 產物存在，再照檔案位置、命令或程式碼修改，最後用本段列出的畫面、指令、log、資料庫或狀態檢查驗證結果。

這一天的事件範例直接接 Day 2 建立的 `ShoppingCart`。不要只做 `console.log()`，而是讓子元件通知父元件修改購物車數量。

```ts
type CartItem = {
  id: number
  name: string
  price: number
  quantity: number
}

function increaseQuantity(items: CartItem[], id: number): CartItem[] {
  return items.map((item) =>
    item.id === id ? { ...item, quantity: item.quantity + 1 } : item
  )
}

function decreaseQuantity(items: CartItem[], id: number): CartItem[] {
  return items
    .map((item) => item.id === id ? { ...item, quantity: item.quantity - 1 } : item)
    .filter((item) => item.quantity > 0)
}
```

Vue 用 `emit('increase', id)`，Svelte 用 callback prop，Angular 用 `output<number>()`。做完後要能驗證：按 `+` 數量增加、按 `-` 到 0 會移除商品、總金額跟著變。

### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- CRUD 的新增、刪除、儲存。
- 使用者點擊按鈕觸發篩選、排序、送出 API。

### 不適合使用的情境

- 需要跨頁共享的狀態，不應全部放在單一元件事件函式裡，應考慮 store 或 service。

### 負面例子 / 錯誤用法

錯誤做法：沒有阻止表單預設送出。

問題：瀏覽器重新整理頁面，狀態遺失，使用者以為新增失敗。

修正方向：Vue 用 `@submit.prevent`，Svelte / Angular 也要明確避免預設刷新或使用框架表單策略。

### Junior 常見誤解

- 「click 和 submit 差不多」：表單語意、鍵盤 Enter、可存取性都會受影響。

### 一句話總結

事件處理是把使用者操作轉成狀態變更的地方。

---

## Day 7：條件渲染

### 這篇文章主要在講什麼

這篇示範當清單有資料時顯示列表，沒有資料時顯示空狀態；也示範編輯模式與非編輯模式的條件 UI。

### 為什麼需要這個概念

真實 UI 會有不同狀態：空資料、載入中、錯誤、編輯中、唯讀、成功。條件渲染讓畫面根據狀態切換。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 7：條件渲染 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 條件狀態 | `items.length`、`isEditing` | 決定畫面分支 |
| 條件語法 | `v-if`、`{#if}`、`@if` | 控制元素是否渲染 |
| 空狀態 | Nothing to see here | 沒有資料時的回饋 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 7：條件渲染 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 判斷 `items` 是否有資料。
2. 有資料時渲染清單。
3. 沒資料時渲染空狀態。
4. 用 `isEditing` 控制表單是否顯示。

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<template v-if="items.length > 0">
  <ul>
    <li v-for="item in items" :key="item.id">{{ item.label }}</li>
  </ul>
</template>
<p v-else>Nothing to see here.</p>
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 空列表提示、權限不足提示、表單編輯模式。
- API loading / error / success 三種畫面切換。

### 不適合使用的情境

- 如果只是切換樣式而不是切換 DOM，可以優先考慮 class binding。

### 負面例子 / 錯誤用法

錯誤做法：空清單時什麼都不顯示。

問題：使用者不知道是資料還沒載入、沒有資料，還是系統壞了。

修正方向：為空狀態、錯誤狀態、載入狀態提供明確 UI。

### Junior 常見誤解

- 「條件渲染只是 if else」：它也會影響元件生命週期、DOM 是否存在與表單狀態是否保留。

### 一句話總結

條件渲染讓 UI 能誠實反映目前資料與互動狀態。

---

## Day 8：動態綁定 CSS 類別與樣式

### 這篇文章主要在講什麼

這篇示範項目已購買時加刪除線，高優先時套用 priority 樣式，並依 checkbox 狀態讓字體變粗。

### 為什麼需要這個概念

UI 不只顯示文字，也要用視覺狀態提示使用者：已完成、錯誤、高優先、不可用、選取中。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 8：動態綁定 CSS 類別與樣式 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 狀態欄位 | `purchased`、`highPriority` | 決定樣式 |
| 類別綁定 | `:class`、`class:`、`[class.xxx]` | 根據狀態套 class |
| 樣式綁定 | `:style`、`style:`、`[style.xxx]` | 根據狀態套 inline style |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 8：動態綁定 CSS 類別與樣式 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 在 item 上保存 UI 相關狀態。
2. 使用 class binding 套用語意 class。
3. 點擊項目時切換 purchased。
4. 用 CSS 控制實際樣式。

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<li
  :class="[{ priority: item.highPriority }, { strikeout: item.purchased }]"
  @click="item.purchased = !item.purchased"
>
  {{ item.label }}
</li>
```

```css
/*
 * 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。
 * 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。
 * 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。
 */
.priority { color: #b45309; }
.strikeout { text-decoration: line-through; color: #6b7280; }
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 任務完成狀態、表單錯誤、警示類型、選取項目。
- 後台列表用顏色表示優先級或狀態。

### 不適合使用的情境

- 不應把重要業務規則只藏在 CSS class，資料模型仍要有明確狀態。

### 負面例子 / 錯誤用法

錯誤做法：用文字內容判斷樣式，例如 label 包含 `urgent` 就套紅色。

問題：資料格式一改，樣式邏輯就壞。

修正方向：使用明確欄位，例如 `highPriority: true`。

### Junior 常見誤解

- 「動態樣式就是 inline style」：多數情況 class binding 更可維護。

### 一句話總結

動態樣式應該從明確狀態推導，而不是從畫面文字猜。

---

## Day 9：屬性綁定

### 這篇文章主要在講什麼

這篇示範當輸入長度不足 5 時，Save Item 按鈕要 disabled。Vue 用 `:disabled`，Svelte 直接綁屬性，Angular 用 property binding。

### 為什麼需要這個概念

屬性綁定讓 HTML 屬性可以由狀態控制，例如 disabled、aria、href、src、title。這會影響互動能力與可存取性。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 9：屬性綁定 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 輸入狀態 | `newItem` | 判斷是否可送出 |
| DOM 屬性 | `disabled` | 控制按鈕是否可用 |
| 條件規則 | `newItem.length < 5` | 基礎驗證條件 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 9：屬性綁定 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 使用輸入狀態保存文字。
2. 定義可送出條件。
3. 將條件綁到按鈕的 disabled 屬性。
4. submit 時仍重新檢查一次。

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<button :disabled="newItem.length < 5" type="submit">
  Save Item
</button>
```

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
const canSave = computed(() => newItem.value.trim().length >= 5)
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 表單驗證、權限控制、載入中禁止重複送出。
- 依資料產生 `href`、圖片 `src`、ARIA 屬性。

### 不適合使用的情境

- 只在前端 disabled 不能當作安全限制，後端仍要驗證。

### 負面例子 / 錯誤用法

錯誤做法：只靠 disabled 防止非法資料。

問題：使用者仍可透過 DevTools 或 API 送出。

修正方向：前端提供體驗，後端提供真正驗證。

### Junior 常見誤解

- 「按鈕 disabled 就安全了」：disabled 是 UX，不是安全邊界。

### 一句話總結

屬性綁定讓 UI 能根據狀態改變行為，但真正規則仍要在資料層驗證。

---

## Day 10：響應式與衍生狀態

### 這篇文章主要在講什麼

這篇介紹從既有狀態推導新狀態：反轉清單、已購買數量、顯示文字。Vue 與 Angular 用 `computed`，Svelte 用 `$derived`。

### 為什麼需要這個概念

很多 UI 值不應重複保存，而應從來源資料推導。這可以避免狀態不同步。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 10：響應式與衍生狀態 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 來源狀態 | `items` | 唯一資料來源 |
| 衍生狀態 | `reversedItems`、`purchasedCount` | 從來源狀態計算 |
| 響應式計算 | `computed`、`$derived` | 來源變更時自動更新 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 10：響應式與衍生狀態 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 保留 `items` 作為唯一來源。
2. 用 computed 建立反轉清單。
3. 用 computed 計算已購買數量。
4. 模板只讀衍生結果。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
const reversedItems = computed(() => [...items.value].reverse())

const purchasedCount = computed(() =>
  items.value.filter((item) => item.purchased).length
)

const purchasedLabel = computed(() =>
  `${purchasedCount.value} ${purchasedCount.value === 1 ? 'item' : 'items'} purchased`
)
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 統計數字、篩選清單、排序清單、金額合計。
- 表單是否可送出、目前使用者是否有權限。

### 不適合使用的情境

- 需要觸發副作用的操作不該放 computed，例如呼叫 API 或寫 localStorage。

### 負面例子 / 錯誤用法

錯誤做法：同時保存 `items` 和 `purchasedCount`，每次新增刪除都手動更新兩者。

問題：一個地方忘記更新，畫面就顯示錯誤統計。

修正方向：能推導的值就用 computed / derived。

### Junior 常見誤解

- 「computed 只是函式」：computed 會追蹤依賴並快取結果，語意上是衍生狀態。

### 一句話總結

衍生狀態應該計算出來，不要手動維護第二份資料。

---

## Day 11：部署到 GitHub Pages

### 這篇文章主要在講什麼

這篇將購物車專案部署到 GitHub Pages，重點包含 Vite base path、build、preview、GitHub Actions workflow 與 Angular 部署設定。

### 為什麼需要這個概念

前端專案在本機能跑不代表部署後能跑。GitHub Pages 通常掛在子路徑，若 base path 錯誤，靜態資源會 404。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 11：部署到 GitHub Pages 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 建置輸出 | `dist` | 部署到 Pages 的靜態檔 |
| 子路徑設定 | `base` | 告訴 Vite 資源路徑前綴 |
| 自動化部署 | GitHub Actions | push 後建置並發布 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 11：部署到 GitHub Pages 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 設定專案的 base path。
2. 執行 build。
3. 本機 preview 確認子路徑可正常載入。
4. 建立 GitHub Actions workflow。
5. 推送後檢查 Pages URL。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
// vite.config.ts
export default defineConfig({
  base: '/fundamental-vue3/',
  plugins: [vue()],
})
```

```yaml
# 範例用途：示範「實作流程與程式碼例子」中的 YAML 設定。
# 參數說明：欄位值、分支名稱、指令與環境變數要依專案實際設定替換。
# 回傳結果 / 副作用：被工具或 CI/CD 載入後，會影響建置、部署或執行流程。
name: Deploy
on:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm run build
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 文件網站、demo app、作品集小專案。
- 教學系列要分享可操作成果。

### 不適合使用的情境

- 需要 server-side runtime、私密 API key 或後端邏輯的系統，不適合只靠 GitHub Pages。

### 負面例子 / 錯誤用法

錯誤做法：本機 `/` 可跑，就直接部署到 `/repo-name/`，沒有設定 base。

問題：JS、CSS、圖片路徑全部指到錯誤位置。

修正方向：依 GitHub Pages repository path 設定 base，並用 preview 驗證。

### Junior 常見誤解

- 「build 成功代表部署成功」：還要確認資源路徑、route fallback、環境變數。

### 一句話總結

部署前要把本機根路徑思維改成 GitHub Pages 的子路徑思維。

---

## Day 12：開始 Vue 元件入門課程

### 這篇文章主要在講什麼

作者開始第二階段課程，準備 Coffee Plan、GitHub Profile、Alert 三組元件練習，並重新建立乾淨專案與全域樣式。

### 為什麼需要這個概念

換一個小專案可以驗證同一套框架概念是否能轉移：從單一購物車元件，進到父子元件、props、slot、資料擷取與動態元件。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 12：開始 Vue 元件入門課程 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 課程階段 | Vue Component Fundamentals | 聚焦元件基礎 |
| 小專案 | Coffee Plan、GitHub Profile、Alert | 用不同場景練習元件設計 |
| 全域樣式 | copied global CSS | 統一 UI 起點 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 12：開始 Vue 元件入門課程 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 建立新專案。
2. 清掉 starter template 中不需要的檔案。
3. 放入課程提供的 CSS。
4. 先確認空白頁面與樣式載入。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
// 建議建立共用型別資料夾，避免後續元件各自定義
export type CoffeePlan = {
  id: number
  name: string
  selected: boolean
}
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 開始新功能前先建立乾淨 module。
- 教學專案每個階段切出獨立練習，避免前面程式碼干擾。

### 不適合使用的情境

- 真實產品不應為每個功能重開專案，應在既有架構內分 module。

### 負面例子 / 錯誤用法

錯誤做法：複製舊專案後不清理無關程式碼。

問題：後續 bug 很難知道來自新功能還是舊殘留。

修正方向：教學或 POC 用乾淨 starter；產品中則用清楚 module 邊界。

### Junior 常見誤解

- 「重開專案很浪費」：學習比較時，乾淨起點能讓差異更清楚。

### 一句話總結

新的練習階段需要乾淨環境，讓元件概念本身浮出來。

---

## Day 13：建立帶有 Prop 的 CoffeePlan 元件

### 這篇文章主要在講什麼

這篇把重複的 coffee plan HTML 抽成 `CoffeePlan` 元件，並透過 `name` prop 顯示不同方案名稱。

### 為什麼需要這個概念

props 是父元件把資料交給子元件的主要方式。它讓同一個子元件能重複使用，但內容由父元件決定。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 13：建立帶有 Prop 的 CoffeePlan 元件 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 子元件 | `CoffeePlan` | 顯示單一咖啡方案 |
| 輸入資料 | `name` prop | 由父元件提供方案名稱 |
| 父元件 | App / PlanPicker | 持有方案清單並建立子元件 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 13：建立帶有 Prop 的 CoffeePlan 元件 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 找出重複 HTML 區塊。
2. 建立 `CoffeePlan` 子元件。
3. 定義 `name` prop。
4. 父元件傳入不同方案名稱。

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<script setup lang="ts">
defineProps<{ name: string }>()
</script>

<template>
  <div class="plan">
    <span class="title">{{ name }}</span>
  </div>
</template>
```

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<CoffeePlan name="The Single" />
<CoffeePlan name="The Curious" />
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 商品卡片、使用者卡片、方案卡片、文章卡片。
- 子元件樣式固定，但內容由父層資料決定。

### 不適合使用的情境

- 子元件不應直接修改 prop；要用事件通知父元件。

### 負面例子 / 錯誤用法

錯誤做法：子元件內硬編所有方案名稱。

問題：元件無法重用，新增方案要改子元件。

修正方向：讓父元件持有資料，子元件只負責顯示單一資料。

### Junior 常見誤解

- 「prop 是子元件自己的資料」：prop 的來源是父元件，子元件應把它視為輸入。

### 一句話總結

props 讓子元件從硬編內容變成可重用的資料顯示器。

---

## Day 14：建立 PlanPicker 父元件

### 這篇文章主要在講什麼

這篇將 coffee plan 清單從 App 抽到 `PlanPicker` 父元件，由 `PlanPicker` 管理清單並渲染多個 `CoffeePlan`。

### 為什麼需要這個概念

當一組子元件共享同一份清單或互動狀態時，需要一個父元件作為協調者，而不是讓根元件越來越肥。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 14：建立 PlanPicker 父元件 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 父元件 | `PlanPicker` | 管理方案清單 |
| 子元件 | `CoffeePlan` | 顯示單一方案 |
| 根元件 | App | 掛載 PlanPicker |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 14：建立 PlanPicker 父元件 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 建立 `PlanPicker`。
2. 將 `plans` 陣列移入 `PlanPicker`。
3. 用清單渲染建立多個 `CoffeePlan`。
4. App 只保留 `<PlanPicker />`。

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<script setup lang="ts">
import { ref } from 'vue'
import CoffeePlan from './CoffeePlan.vue'

const plans = ref(['The Single', 'The Curious', 'The Addict'])
</script>

<template>
  <div class="plans">
    <CoffeePlan v-for="plan in plans" :key="plan" :name="plan" />
  </div>
</template>
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 卡片列表、步驟選擇器、Tab 列表、選項清單。
- 父元件要負責排序、篩選、選取狀態。

### 不適合使用的情境

- 如果父元件只轉傳資料而沒有協調責任，可能只是多一層包裝。

### 負面例子 / 錯誤用法

錯誤做法：App 同時管理所有資料、所有事件、所有子元件。

問題：根元件責任模糊，功能一多就難測試。

修正方向：把有共同上下文的子元件移到專門父元件。

### Junior 常見誤解

- 「父元件一定是 App」：父元件只是相對概念，任何組合子元件的元件都可以是父元件。

### 一句話總結

父元件的價值在協調一組相關子元件與它們共享的狀態。

---

## Day 15：新增 Coffee Plan 表單

### 這篇文章主要在講什麼

這篇新增 `AddCoffeePlan` 子元件，讓使用者輸入新方案名稱，並通知 `PlanPicker` 把方案加進清單。

### 為什麼需要這個概念

表單子元件通常不應直接改父層清單，而是透過 emit / callback / output 通知父元件。這能維持資料流清楚。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 15：新增 Coffee Plan 表單 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 表單子元件 | `AddCoffeePlan` | 收集新方案名稱 |
| 父元件 | `PlanPicker` | 實際更新方案清單 |
| 自訂事件 | `newCoffeePlan` | 子元件通知父元件新增資料 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 15：新增 Coffee Plan 表單 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 建立 `AddCoffeePlan`。
2. 在子元件保存暫時輸入值。
3. submit 時 emit 新方案名稱。
4. 父元件收到事件後更新 `plans`。

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<script setup lang="ts">
import { ref } from 'vue'

const emit = defineEmits<{ newCoffeePlan: [name: string] }>()
const newPlan = ref('')

function addPlan() {
  if (!newPlan.value.trim()) return
  emit('newCoffeePlan', newPlan.value.trim())
  newPlan.value = ''
}
</script>

<template>
  <form @submit.prevent="addPlan">
    <input v-model="newPlan" />
    <button type="submit">Add</button>
  </form>
</template>
```


### 主線落地：AddCoffeePlan 必須接回 PlanPicker

範例規則：這段範例必須看成當天 Day 的完整範圍，不是孤立程式碼；先確認前置檔案或前一 Day 產物存在，再照檔案位置、命令或程式碼修改，最後用本段列出的畫面、指令、log、資料庫或狀態檢查驗證結果。

這一天的表單不是孤立表單，它要把新方案送回 `PlanPicker`，讓 Day 13 的 `CoffeePlan` 和 Day 14 的清單一起更新。

```vue
<script setup lang="ts">
import { ref } from 'vue'
import CoffeePlan from './CoffeePlan.vue'
import AddCoffeePlan from './AddCoffeePlan.vue'

const plans = ref(['The Single', 'The Curious'])
const selectedPlan = ref<string | null>(null)

function addPlan(name: string) {
  const value = name.trim()
  if (value !== '' && !plans.value.includes(value)) {
    plans.value.push(value)
  }
}
</script>

<template>
  <AddCoffeePlan @new-coffee-plan="addPlan" />
  <CoffeePlan
    v-for="plan in plans"
    :key="plan"
    :name="plan"
    :active="plan === selectedPlan"
    @select="selectedPlan = plan"
  />
</template>
```

`AddCoffeePlan` 只處理輸入與送出，`CoffeePlan` 只顯示與選取，`PlanPicker` 保存清單與目前選取狀態。

### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 新增商品分類、標籤、方案、留言。
- 子元件收資料，父元件決定如何保存。

### 不適合使用的情境

- 若新增流程需要 API、驗證、權限，父元件可能也不夠，應移到 service / action。

### 負面例子 / 錯誤用法

錯誤做法：子元件直接 import 父元件的陣列或全域變數來 push。

問題：資料流隱性耦合，測試與重用困難。

修正方向：子元件 emit 意圖，父元件決定狀態變更。

### Junior 常見誤解

- 「emit 是多此一舉」：emit 讓資料變更方向清楚，是元件重用的基礎。

### 一句話總結

表單子元件負責收集輸入，父元件負責改變清單。

---

## Day 16：使用元件事件選擇咖啡方案

### 這篇文章主要在講什麼

這篇讓 `CoffeePlan` 被點擊時通知 `PlanPicker`，父元件保存目前 active plan，並讓被選方案加上視覺狀態。

### 為什麼需要這個概念

多個兄弟元件之間通常不能直接互相改狀態，應把共享狀態提升到共同父元件。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 16：使用元件事件選擇咖啡方案 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 子元件事件 | `selectedPlan` | 回報使用者選了哪個方案 |
| 共享狀態 | active plan | 父元件保存目前選取 |
| 視覺狀態 | selected border / inactive | 根據 active plan 顯示 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 16：使用元件事件選擇咖啡方案 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 子元件點擊時 emit `selectedPlan`。
2. 父元件接收方案名稱。
3. 父元件更新 `activePlan`。
4. 父元件把 `isSelected` 傳回子元件。

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<!-- CoffeePlan.vue -->
<script setup lang="ts">
const props = defineProps<{ name: string; selected: boolean }>()
const emit = defineEmits<{ selectedPlan: [name: string] }>()
</script>

<template>
  <button :class="{ selected }" @click="emit('selectedPlan', props.name)">
    {{ name }}
  </button>
</template>
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 單選卡片、方案選擇、Tab、步驟精靈。
- 子元件互斥狀態，例如只有一個展開。

### 不適合使用的情境

- 選取狀態需要跨頁共享時，父元件狀態可能不夠，應考慮 route、store 或 URL query。

### 負面例子 / 錯誤用法

錯誤做法：每個 `CoffeePlan` 自己保存 `selected`，彼此不知道其他元件狀態。

問題：可能同時有多個方案被選中。

修正方向：把 active state 提升到 `PlanPicker`。

### Junior 常見誤解

- 「兄弟元件可以互相呼叫」：通常應透過共同父元件或共享狀態協調。

### 一句話總結

兄弟元件共享的狀態，應該交給共同父元件管理。

---

## Day 17：在 HTML 模板中渲染動態內容

### 這篇文章主要在講什麼

這篇比較 Vue slot、Svelte snippet / render 與 Angular `ng-content` / `ng-template`，用來把父元件提供的內容投影到子元件中。

### 為什麼需要這個概念

有些元件只知道「版位」，不知道內容。像按鈕文字、卡片 actions、選中圖示，應由使用端決定，元件提供插槽位置。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 17：在 HTML 模板中渲染動態內容 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 內容投影 | slot / snippet / ng-content | 父元件提供模板內容 |
| 插槽屬性 | slot props | 子元件把狀態傳回父層模板 |
| 模板片段 | render / ng-template | 延後渲染的 UI 片段 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 17：在 HTML 模板中渲染動態內容 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 子元件提供 slot 位置。
2. 子元件可把 hover / selected 狀態作為 slot props。
3. 父元件決定實際顯示文字或圖示。
4. 驗證預設內容與投影內容都能工作。

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<!-- AddCoffeePlan.vue -->
<button @mouseenter="hover = true" @mouseleave="hover = false">
  <slot name="btn" :hover="hover">Add</slot>
</button>
```

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<!-- 使用端 -->
<template #btn="{ hover }">
  {{ hover ? 'Add this plan' : 'Add' }}
</template>
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 通用 Button、Modal、Card、Table cell、Toolbar。
- 元件控制結構，使用端控制內容。

### 不適合使用的情境

- 如果內容永遠固定，不需要 slot，直接寫在子元件即可。

### 負面例子 / 錯誤用法

錯誤做法：為每種按鈕文字做一個 prop，例如 `hoverText`、`selectedText`、`iconName` 越加越多。

問題：元件 API 膨脹，仍然無法涵蓋複雜內容。

修正方向：需要開放內容時使用 slot / snippet / template。

### Junior 常見誤解

- 「slot 只是把 HTML 塞進去」：slot 是元件抽象的一部分，可讓結構與內容分離。

### 一句話總結

內容投影讓元件保留結構控制權，同時把內容決策交給使用端。

---

## Day 18：GitHub Card 專案資料擷取

### 這篇文章主要在講什麼

這篇開始 GitHub 個人檔案卡片專案，重點是建立 GitHub token、環境變數、`GithubProfile` 型別，並在三個框架中用各自方式擷取資料。

### 為什麼需要這個概念

前端常需要呼叫外部 API。資料擷取不能只會 `fetch`，還要處理 token、型別、非同步狀態與不同框架的資料載入模式。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 18：GitHub Card 專案資料擷取 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| API | GitHub API | 提供使用者 profile |
| 認證資料 | GitHub Personal Token | 提高 API 存取能力 |
| 型別 | `GithubProfile` | 定義 API 回傳資料形狀 |
| 資料擷取封裝 | Vue composable、Svelte load、Angular service/httpResource | 封裝 API 呼叫 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 18：GitHub Card 專案資料擷取 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 建立 `.env.example`，不要提交真 token。
2. 定義 `GithubProfile` 型別。
3. 封裝 fetch 函式。
4. 元件只呼叫封裝，不直接散落 API 細節。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
export type GithubProfile = {
  login: string
  name: string
  avatar_url: string
  html_url: string
}

export async function fetchGithubProfile(username: string, token: string) {
  const response = await fetch(`https://api.github.com/users/${username}`, {
    headers: { Authorization: `Bearer ${token}` },
  })
  if (!response.ok) throw new Error('Failed to load GitHub profile')
  return response.json() as Promise<GithubProfile>
}
```


### 主線落地：資料擷取要有 loading / error / ready

範例規則：這段範例必須看成當天 Day 的完整範圍，不是孤立程式碼；先確認前置檔案或前一 Day 產物存在，再照檔案位置、命令或程式碼修改，最後用本段列出的畫面、指令、log、資料庫或狀態檢查驗證結果。

GitHub Card 的 fetch 不要直接塞在畫面中。先定義資料形狀與頁面狀態，Day 19 的元件組合、Day 20 的樣式、Day 21 的部署都接這個狀態。

```ts
type GithubProfile = {
  login: string
  name: string | null
  avatar_url: string
  html_url: string
  public_repos: number
  followers: number
}

type GithubProfileState =
  | { status: 'loading' }
  | { status: 'error'; message: string }
  | { status: 'ready'; profile: GithubProfile }

async function loadGithubProfile(username: string): Promise<GithubProfile> {
  const response = await fetch(`https://api.github.com/users/${username}`)
  if (!response.ok) throw new Error(`GitHub API failed: ${response.status}`)
  return await response.json() as GithubProfile
}
```

驗證時要故意輸入不存在的 username，確認 error 畫面會出現，而不是整頁空白。

### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 使用第三方 API 建立 profile card、dashboard、搜尋功能。
- 將資料擷取邏輯抽到 composable / service / loader。

### 不適合使用的情境

- 私密 token 不應暴露在純前端公開網站；需要後端代理或 GitHub Actions build secret。

### 負面例子 / 錯誤用法

錯誤做法：把 GitHub token 寫死在程式碼並 commit。

問題：token 外洩，可能被濫用。

修正方向：使用 `.env`，公開 repo 只提交 `.env.example`；真正敏感操作放後端。

### Junior 常見誤解

- 「VITE_ 開頭環境變數很安全」：Vite 前端環境變數會被打包到瀏覽器，可被看見。

### 一句話總結

前端 API 擷取要同時處理資料型別、非同步流程與 token 暴露風險。

---

## Day 19：GitHub Card 元件組合

### 這篇文章主要在講什麼

這篇建立 `GithubProfileList` 與 `GithubProfileCard`，前者迭代 username 清單，後者負責載入並顯示單一 profile。

### 為什麼需要這個概念

列表與卡片是常見組合：容器元件負責資料集合，項目元件負責單筆資料呈現。這能讓 UI 結構更清楚。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 19：GitHub Card 元件組合 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 容器元件 | `GithubProfileList` | 持有 username 清單並建立卡片 |
| 展示元件 | `GithubProfileCard` | 顯示單一使用者 |
| 資料來源 | username prop | 決定要抓哪位使用者 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 19：GitHub Card 元件組合 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 在 List 元件準備 username 陣列。
2. 用清單渲染建立多個 Card。
3. Card 接收 `username` prop。
4. Card 內部呼叫 profile 擷取邏輯。

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<script setup lang="ts">
import GithubProfileCard from './GithubProfileCard.vue'

const usernames = ['yyx990803', 'angular', 'sveltejs']
</script>

<template>
  <GithubProfileCard
    v-for="username in usernames"
    :key="username"
    :username="username"
  />
</template>
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 商品列表與商品卡、使用者列表與使用者卡、儀表板 widget。

### 不適合使用的情境

- 若每張卡都各自打 API，數量大時應改由父層批次載入或加入快取。

### 負面例子 / 錯誤用法

錯誤做法：List 元件同時包含卡片所有 HTML、API、錯誤處理、樣式。

問題：元件過大，不容易替換卡片樣式或測試單一卡片。

修正方向：List 管集合，Card 管單筆呈現。

### Junior 常見誤解

- 「拆元件只為了少寫 HTML」：拆元件更重要的是責任分離與重用。

### 一句話總結

列表元件管集合，卡片元件管單筆資料，是前端組合的基本分工。

---

## Day 20：GitHub Card 樣式設計

### 這篇文章主要在講什麼

這篇安裝 Tailwind CSS 與 DaisyUI，將 GitHub profile card 改成更完整的卡片 UI，並以 utility classes 取代大量自訂 CSS。

### 為什麼需要這個概念

樣式框架能加速 UI 實作，但前提是資料結構與元件分工已經清楚。樣式不應掩蓋元件責任。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 20：GitHub Card 樣式設計 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| Utility CSS | Tailwind CSS | 用 class 快速組合樣式 |
| UI 元件樣式 | DaisyUI card | 提供預設卡片外觀 |
| Profile 資料 | `profile` | 填入卡片內容 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 20：GitHub Card 樣式設計 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 安裝 Tailwind 與 DaisyUI。
2. 將插件加入 CSS / build 設定。
3. 先套靜態 card layout。
4. 再將假資料替換成 `profile` 欄位。

```html
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<div class="card bg-base-100 shadow-sm">
  <figure>
    <img :src="profile.avatar_url" :alt="profile.login" />
  </figure>
  <div class="card-body">
    <h2 class="card-title">{{ profile.name }}</h2>
    <a :href="profile.html_url">{{ profile.login }}</a>
  </div>
</div>
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 快速建立 demo、後台卡片、個人檔案、狀態面板。

### 不適合使用的情境

- 大型產品若沒有設計規範，直接堆 utility class 可能造成 UI 不一致。

### 負面例子 / 錯誤用法

錯誤做法：把所有業務狀態都用 class 名稱表示，不保留資料欄位。

問題：樣式與資料語意混在一起，之後很難改版或測試。

修正方向：資料模型先清楚，再將資料映射到 class。

### Junior 常見誤解

- 「用了 Tailwind 就不用設計」：Tailwind 是工具，不會自動產生資訊架構。

### 一句話總結

樣式框架能加速呈現，但元件分工與資料語意仍然要先做好。

---

## Day 21：部署 GitHub Profile 專案

### 這篇文章主要在講什麼

這篇將 GitHub Profile 專案部署到 GitHub Pages，並特別提到含敏感環境變數時，Angular 不能只靠簡單 `ng deploy`，需要透過 GitHub Actions 注入 secrets。

### 為什麼需要這個概念

部署含 API token 的前端專案時，要分清楚 build-time 變數與 runtime 安全。公開前端無法真正保護秘密。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 21：部署 GitHub Profile 專案 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 部署平台 | GitHub Pages | 發布靜態網站 |
| CI/CD | GitHub Actions | build 並部署 |
| Secret | GitHub token / environment variable | 建置時提供設定 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 21：部署 GitHub Profile 專案 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 設定 Vite / Angular 的 base path。
2. 將 token 放到 GitHub repository secrets。
3. 在 Actions build 時注入環境變數。
4. 部署後測試 API 是否正常。

```yaml
# 範例用途：示範「實作流程與程式碼例子」中的 YAML 設定。
# 參數說明：欄位值、分支名稱、指令與環境變數要依專案實際設定替換。
# 回傳結果 / 副作用：被工具或 CI/CD 載入後，會影響建置、部署或執行流程。
env:
  VITE_GITHUB_TOKEN: ${{ secrets.GH_PROFILE_TOKEN }}

steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
  - run: npm ci
  - run: npm run build
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- Demo 網站需要讀公開 API，但有 rate limit token。
- 作品集專案需要自動部署。

### 不適合使用的情境

- 真正私密 API key 不應放在前端網站，即使透過 Actions 注入，build 後仍可能暴露。

### 負面例子 / 錯誤用法

錯誤做法：以為 GitHub Secrets 注入前端後，瀏覽器看不到。

問題：前端 bundle 仍可能包含該值。

修正方向：敏感操作放 server；前端只使用可公開的設定。

### Junior 常見誤解

- 「CI secret 等於前端 secret」：CI secret 只保護建置過程，不保證瀏覽器端安全。

### 一句話總結

部署前端時，要清楚哪些設定會進入瀏覽器，哪些必須留在伺服器。

---

## Day 22：Alert List 與 Alert Component

### 這篇文章主要在講什麼

這篇開始 Alert 元件專案，建立 Alert list 與單一 Alert component，並用 DaisyUI / Tailwind 顯示不同類型警示，同時比較 Vue `defineModel`、Svelte `$bindable`、Angular `model` 的雙向綁定。

### 為什麼需要這個概念

Alert 是常見 UI，但它包含資料清單、類型、關閉狀態、樣式與父子同步。小元件也能練到完整資料流。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 22：Alert List 與 Alert Component 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 容器 | Alert List | 迭代警示資料 |
| 子元件 | Alert Component | 顯示單一警示 |
| 警示型別 | info、success、warning、error | 決定樣式與語意 |
| 雙向狀態 | `defineModel`、`$bindable`、`model` | 父子同步關閉狀態 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 22：Alert List 與 Alert Component 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 定義 alert type。
2. Alert List 保存警示清單。
3. Alert Component 接收 type 與 message。
4. 關閉時回報父層更新狀態。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
type AlertType = 'info' | 'success' | 'warning' | 'error'

type AlertMessage = {
  id: number
  type: AlertType
  text: string
}
```

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<div class="alert" :class="`alert-${alert.type}`">
  <span>{{ alert.text }}</span>
  <button @click="close(alert.id)">Close</button>
</div>
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 表單成功 / 失敗訊息、系統通知、後台狀態提示。

### 不適合使用的情境

- Alert 不應用來承載需要使用者長時間操作的複雜流程；那應該是 modal 或頁面狀態。

### 負面例子 / 錯誤用法

錯誤做法：每種 alert type 寫一個完全不同元件，內容幾乎一樣。

問題：重複維護，新增欄位要改四份。

修正方向：用資料 type 決定樣式與圖示。

### Junior 常見誤解

- 「小 UI 不需要資料模型」：Alert 一旦有 type、close、reopen，就需要清楚模型。

### 一句話總結

Alert 元件雖小，但很適合練習清單、型別、樣式與父子同步。

---

## Day 23：動態渲染 SVG 圖示

### 這篇文章主要在講什麼

這篇為不同 alert type 建立 SVG icon component，並用動態元件方式決定要渲染哪個 icon。Vue 用 `:is`，Svelte 用大寫動態 component，Angular 用 `ngComponentOutlet`。

### 為什麼需要這個概念

當 UI 類型會擴充時，不應在模板寫一大串 if/else。動態元件讓「type 對應 component」變成可維護的映射表。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 23：動態渲染 SVG 圖示 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 圖示元件 | `InfoIcon`、`SuccessIcon`、`WarningIcon`、`ErrorIcon` | 顯示不同 SVG |
| 動態渲染 | `:is`、dynamic component、`ngComponentOutlet` | 根據 type 選元件 |
| 映射表 | icon map | 將 alert type 對應到圖示 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 23：動態渲染 SVG 圖示 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 建立每個 icon component。
2. 建立 `iconByType` 映射。
3. Alert Component 根據 type 取出 icon。
4. 渲染對應 component。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
const iconByType = {
  info: InfoIcon,
  success: SuccessIcon,
  warning: WarningIcon,
  error: ErrorIcon,
} as const
```

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<component :is="iconByType[alert.type]" />
<span>{{ alert.text }}</span>
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 表單欄位型態對應不同 input component。
- 儀表板 widget type 對應不同圖表。
- 通知 type 對應不同 icon。

### 不適合使用的情境

- 類型很少且不會擴充時，簡單條件渲染也可以。

### 負面例子 / 錯誤用法

錯誤做法：模板中為每個 type 寫一段重複 alert markup。

問題：修改樣式時要改多份，容易漏。

修正方向：共同結構保留一份，只有 icon 用動態元件替換。

### Junior 常見誤解

- 「動態元件很進階，所以不要用」：當 type-to-component 映射清楚時，它反而讓模板更簡單。

### 一句話總結

動態元件適合處理「同一版位，不同實作元件」的場景。

---

## Day 24：新增 Alert Bar 更換樣式

### 這篇文章主要在講什麼

這篇建立 Alert Bar，用來控制是否顯示關閉按鈕、改變 alert 樣式與排列方向，並讓這些設定影響 Alert List。

### 為什麼需要這個概念

當一個控制列會影響一組元件時，需要清楚定義「控制狀態」與「被控制 UI」之間的資料流。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 24：新增 Alert Bar 更換樣式 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 控制元件 | Alert Bar | 提供樣式與顯示設定 |
| 被控制元件 | Alert List / Alert | 依設定改變呈現 |
| 雙向設定 | selected style / direction / close button | 父子之間同步的 UI 設定 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 24：新增 Alert Bar 更換樣式 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 定義 alert UI settings。
2. Alert Bar 修改 settings。
3. Alert List 接收 settings。
4. Alert Component 根據 settings 套 class 或顯示按鈕。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
type AlertSettings = {
  showCloseButton: boolean
  direction: 'vertical' | 'horizontal'
  style: 'soft' | 'outline' | 'solid'
}
```

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<button v-if="settings.showCloseButton" @click="close(alert.id)">
  Close
</button>
```


### 主線落地：AlertBar 改狀態，不直接改 DOM

範例規則：這段範例必須看成當天 Day 的完整範圍，不是孤立程式碼；先確認前置檔案或前一 Day 產物存在，再照檔案位置、命令或程式碼修改，最後用本段列出的畫面、指令、log、資料庫或狀態檢查驗證結果。

AlertBar 要控制 alert 顯示設定，而不是用 DOM query 去改 class。先定義設定與關閉狀態，Day 25 的重開與 Day 26 的抽取都接這裡。

```ts
type AlertType = 'info' | 'success' | 'warning' | 'error'

type AlertSettings = {
  dismissible: boolean
  layout: 'stacked' | 'inline'
}

function visibleAlerts<T extends { type: AlertType }>(alerts: T[], closedTypes: AlertType[]) {
  return alerts.filter((alert) => !closedTypes.includes(alert.type))
}
```

做完後要能驗證：切換 layout 會影響整個 AlertList；關閉某類 alert 後，Day 25 可以從狀態把它重開。

### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 後台列表工具列控制顯示密度、排序、篩選。
- 設定面板控制預覽區外觀。

### 不適合使用的情境

- 如果設定只影響單一元件，沒有必要抽成獨立 bar。

### 負面例子 / 錯誤用法

錯誤做法：Alert Bar 直接用 DOM query 修改 Alert class。

問題：繞過框架狀態，UI 與資料不同步。

修正方向：Alert Bar 修改狀態，Alert 根據狀態渲染。

### Junior 常見誤解

- 「改 class 用 querySelector 比較快」：在框架中直接改 DOM 容易和 render 結果打架。

### 一句話總結

控制列應該改變狀態，而不是直接操控其他元件的 DOM。

---

## Day 25：重新開啟已關閉的 Alert

### 這篇文章主要在講什麼

這篇擴充 Alert Bar，追蹤已關閉的 alert type，並提供重新開啟全部或特定類型 alert 的按鈕。

### 為什麼需要這個概念

關閉 UI 不只是把元素移除，還要知道「哪些被關閉」與「如何恢復」。這是狀態集合管理問題。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 25：重新開啟已關閉的 Alert 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 關閉狀態 | `closedNotifications` | 記錄已關閉 alert type |
| 重開操作 | Open all / open type | 從關閉集合移除項目 |
| 雙向綁定 | `defineModel`、`$bindable`、`model` | Alert Bar 與父層同步 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 25：重新開啟已關閉的 Alert 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 建立 `closedNotifications` 陣列。
2. 關閉 alert 時加入 type。
3. Alert List 過濾已關閉 type。
4. Alert Bar 顯示重開按鈕。
5. 點擊重開時從陣列移除。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
const closedNotifications = ref<string[]>([])

function close(type: string) {
  if (!closedNotifications.value.includes(type)) {
    closedNotifications.value.push(type)
  }
}

function reopen(type: string) {
  closedNotifications.value = closedNotifications.value.filter((x) => x !== type)
}
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 可暫時隱藏的通知、篩選條件 chip、可恢復的面板。

### 不適合使用的情境

- 重要錯誤訊息不應讓使用者永久關閉而沒有其他入口。

### 負面例子 / 錯誤用法

錯誤做法：關閉 alert 後直接從原始資料刪除。

問題：無法知道要恢復什麼，重新開啟功能做不出來。

修正方向：保留原始資料，用 `closedNotifications` 表示目前隱藏狀態。

### Junior 常見誤解

- 「畫面不顯示就等於資料不存在」：顯示狀態與資料來源要分開。

### 一句話總結

可恢復的關閉行為，應該保存隱藏狀態，而不是刪掉原始資料。

---

## Day 26：從 AlertBar 抽取邏輯和元件

### 這篇文章主要在講什麼

這篇重構 AlertBar：將重複的 label + select 抽成 `AlertDropdown`，並把 `closedNotifications` 相關邏輯封裝到 Vue composable、Svelte store 或 Angular service。

### 為什麼需要這個概念

重構不是為了漂亮，而是當元件同時負責 UI 控制、狀態管理、資料操作時，要拆出可命名的責任。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 26：從 AlertBar 抽取邏輯和元件 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| UI 子元件 | `AlertDropdown` | 封裝 label + select |
| 狀態管理 | Vue composable、Svelte store、Angular service | 封裝關閉 / 重開通知邏輯 |
| 原元件 | `AlertBar` | 組合 dropdown 與操作按鈕 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 26：從 AlertBar 抽取邏輯和元件 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 找出 AlertBar 中重複的 select 結構。
2. 抽成 `AlertDropdown`，用 props 接收 label 與 items。
3. 把關閉 / 重開邏輯抽成 composable / store / service。
4. AlertBar 只負責組合 UI 與呼叫方法。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
export function useClosedNotifications() {
  const closed = ref<string[]>([])
  const close = (type: string) => {
    if (!closed.value.includes(type)) closed.value.push(type)
  }
  const reopen = (type: string) => {
    closed.value = closed.value.filter((x) => x !== type)
  }
  const reopenAll = () => {
    closed.value = []
  }
  return { closed, close, reopen, reopenAll }
}
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 多元件共用同一組通知狀態。
- 將複雜 UI 控制列拆成小元件。

### 不適合使用的情境

- 邏輯只出現一次且很短時，過早抽象會讓追 code 變困難。

### 負面例子 / 錯誤用法

錯誤做法：一看到函式超過十行就抽 service。

問題：抽象名稱不清楚，反而分散閱讀成本。

修正方向：以「責任是否可命名、是否被重用、是否降低耦合」決定是否抽取。

### Junior 常見誤解

- 「重構就是拆檔案」：重構的目標是讓責任更清楚，不只是讓檔案變多。

### 一句話總結

好的抽取會讓元件更像組合者，讓狀態邏輯有自己的家。

---

## Day 27：建立簡單部落格頁面

### 這篇文章主要在講什麼

這篇開始 Blog 專案，從 JSONPlaceholder 取得 posts，建立文章列表與文章詳情路由，並準備後續取得作者資料與 loading 狀態。

### 為什麼需要這個概念

真實應用通常不是單一畫面，而是列表頁與詳情頁的組合。路由與資料擷取會共同決定使用者如何瀏覽資料。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 27：建立簡單部落格頁面 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| API | JSONPlaceholder posts | 提供文章資料 |
| 型別 | `Post`、`User` | 定義文章與作者資料 |
| 路由 | Home、Post detail | 列表頁與詳情頁 |
| 資料封裝 | `usePost` composable / service / load | 擷取文章資料 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 27：建立簡單部落格頁面 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 定義 `Post` 與 `User` 型別。
2. 建立文章列表路由。
3. 建立文章詳情路由。
4. 封裝 `fetchPosts` 與 `fetchPost`。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
export type Post = {
  userId: number
  id: number
  title: string
  body: string
}

export async function fetchPosts() {
  const response = await fetch('https://jsonplaceholder.typicode.com/posts')
  if (!response.ok) throw new Error('Failed to load posts')
  return response.json() as Promise<Post[]>
}
```


### 主線落地：列表與詳情頁共用資料模型

範例規則：這段範例必須看成當天 Day 的完整範圍，不是孤立程式碼；先確認前置檔案或前一 Day 產物存在，再照檔案位置、命令或程式碼修改，最後用本段列出的畫面、指令、log、資料庫或狀態檢查驗證結果。

Blog 不只是 fetch posts。Day 27 要先讓列表與詳情頁共用資料模型，Day 28 再把作者資料組成畫面需要的 view model。

```ts
type Post = {
  id: number
  userId: number
  title: string
  body: string
}

type PageState<T> =
  | { status: 'loading' }
  | { status: 'error'; message: string }
  | { status: 'ready'; data: T }
```

Day 29 加 loader/error 時，不要臨時在模板塞一堆布林值，直接沿用 `PageState<T>`。重新整理詳情頁也要能載入資料，才算路由與資料流接起來。

### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 部落格、商品目錄、訂單列表與訂單詳情。
- 路由參數決定要載入哪筆資料。

### 不適合使用的情境

- 資料量大時，不應一次載入所有文章，應加入分頁或伺服器查詢。

### 負面例子 / 錯誤用法

錯誤做法：詳情頁從列表頁傳整個 post 物件，重新整理後就沒有資料。

問題：URL 無法獨立代表頁面狀態。

修正方向：詳情頁用 route id 自行載入資料。

### Junior 常見誤解

- 「路由只是換畫面」：路由也應該描述資料識別，例如 `/post/:id`。

### 一句話總結

列表與詳情頁的核心，是讓 URL 能穩定對應到資料。

---

## Day 28：取得貼文作者

### 這篇文章主要在講什麼

這篇使用 post 的 `userId` 呼叫 users 端點取得作者名稱。Vue 用 `useUser` composable，Angular 用 user service / httpResource，SvelteKit 在 load 中新增使用者請求。

### 為什麼需要這個概念

前端常要處理關聯資料：文章有作者、訂單有客戶、留言有使用者。這需要根據一筆資料的外鍵再取得另一筆資料。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 28：取得貼文作者 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 主資料 | `Post` | 包含 `userId` |
| 關聯資料 | `User` | 提供作者名稱 |
| 載入函式 | `useUser` / service / load | 根據 userId 取得使用者 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 28：取得貼文作者 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 載入單篇 post。
2. 從 post 取得 `userId`。
3. 呼叫 users endpoint。
4. 將作者名稱顯示在詳情頁。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
export async function fetchUser(userId: number, signal?: AbortSignal) {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/users/${userId}`,
    { signal },
  )
  if (!response.ok) throw new Error('Failed to load user')
  return response.json() as Promise<User>
}
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 顯示訂單客戶、文章作者、活動建立者。
- 詳情頁需要把多個 API 資料組合成完整畫面。

### 不適合使用的情境

- 如果列表有大量資料，逐筆打 API 會造成 N+1 問題，應考慮批次 API 或後端聚合。

### 負面例子 / 錯誤用法

錯誤做法：每次 render 都重新呼叫使用者 API。

問題：造成大量重複請求與閃爍。

修正方向：用 watch / derived / loader 明確在 `userId` 改變時載入，並加快取或取消舊請求。

### Junior 常見誤解

- 「有 userId 就可以直接顯示作者」：userId 是關聯鍵，不是使用者名稱。

### 一句話總結

關聯資料載入要跟著主資料 ID 變化，而不是跟著畫面 render 次數亂跑。

---

## Day 29：新增 Loader 與 Error 狀態

### 這篇文章主要在講什麼

這篇加入 loading 與 error 顯示。Vue 使用 VueUse `useFetch`，SvelteKit 使用 navigating 與 error helper，Angular 使用 `httpResource` 內建狀態。

### 為什麼需要這個概念

非同步資料不是只有成功。好的 UI 要明確呈現載入中、失敗、空資料與成功狀態，否則使用者不知道系統發生什麼事。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 29：新增 Loader 與 Error 狀態 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 載入狀態 | `isFetching` / navigating / resource loading | 表示請求尚未完成 |
| 錯誤狀態 | `error` | 保存失敗資訊 |
| 成功資料 | `posts` / `post` / `user` | 請求成功後的結果 |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 29：新增 Loader 與 Error 狀態 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 發出資料請求。
2. 請求中顯示 loader。
3. 失敗時顯示錯誤訊息。
4. 成功時顯示資料。
5. 不要讓三種狀態互相重疊。

```vue
<!-- 範例用途：示範「實作流程與程式碼例子」中的畫面模板、元件結構或樣式設定。 -->
<!-- 參數說明：屬性、事件、class、props 或綁定值要依實際元件資料與狀態帶入。 -->
<!-- 回傳結果 / 副作用：會影響畫面渲染、互動行為、樣式呈現或元件對外輸出的事件。 -->
<script setup lang="ts">
import { useFetch } from '@vueuse/core'

const { data: posts, isFetching, error } =
  useFetch<Post[]>('https://jsonplaceholder.typicode.com/posts').json()
</script>

<template>
  <div v-if="isFetching">Loading ...</div>
  <div v-else-if="error">{{ error }}</div>
  <PostCard v-else v-for="post in posts" :key="post.id" :post="post" />
</template>
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- API 列表、搜尋結果、詳情頁、儀表板資料。

### 不適合使用的情境

- 不應用單一全頁 loading 遮住所有局部資料；大型頁面可用區塊級 loading。

### 負面例子 / 錯誤用法

錯誤做法：API 失敗時只在 console log，不顯示任何 UI。

問題：使用者只看到空白畫面，不知道可否重試。

修正方向：畫面上提供錯誤訊息與重試入口。

### Junior 常見誤解

- 「catch error 就夠了」：捕捉錯誤只是程式層，還要轉成使用者看得懂的 UI 狀態。

### 一句話總結

非同步 UI 至少要設計 loading、error、success 三種狀態。

---

## Day 30：比賽結束與總結

### 這篇文章主要在講什麼

作者回顧 30 天用 Vue 3、Svelte 5 / SvelteKit 與 Angular 完成五組示範：Shopping Cart、Coffee Plan、GitHub Profile、Alert、Blog。結論是三個框架差異沒有想像中巨大，正確的基礎概念更重要。

### 為什麼需要這個概念

學框架最怕只記 API，不知道自己在解什麼問題。系列結束時回顧專案，可以把零散技巧收斂成可轉移能力。


### 學完這篇你應該會做到什麼

- 能用 Angular 開發者熟悉的角度，說明 Day 30：比賽結束與總結 的核心觀念。
- 能照著範例完成對應的 Vue / Svelte 元件、狀態或部署流程。
- 能用畫面、測試資料、瀏覽器 console 或建置結果驗證自己做對了。
### 定義名稱對照表

| 角色 | 文章中的名稱 | 功能 / 責任 |
| --- | --- | --- |
| 示範 1 | Shopping Cart | 單元件、事件、清單、狀態 |
| 示範 2 | Coffee Plan | 父子元件、props、事件、內容投影 |
| 示範 3 | GitHub Profile | API 擷取、元件組合、部署 |
| 示範 4 | Alert | 動態元件、雙向綁定、狀態重構 |
| 示範 5 | Blog | 路由、關聯資料、loading / error |


### 實作任務情境

把這篇當成一個小任務：你要在既有前端專案中完成 Day 30：比賽結束與總結 對應的功能，並比較 Angular、Vue 3、Svelte 5 在寫法與心智模型上的差異。

### 操作前檢查

- 先確認你知道本篇是在 Vue 3、Svelte 5，還是部署 / 工具流程中操作。
- 若有程式碼，先確認檔案應放在哪個 component、page、route 或 workflow 設定檔。
- 若有 CLI 或 YAML，先在測試專案或分支操作，不要直接改正式部署設定。
### 範例形式與實作

完整流程：

1. 列出每個專案學到的核心概念。
2. 把框架 API 對照到共同概念。
3. 回頭檢查哪些概念能跨框架轉移。
4. 針對不熟的部分做下一輪小練習。

```ts
// 範例用途：示範「實作流程與程式碼例子」中的程式流程或 API 使用方式。
// 參數說明：方法參數、DTO、URL、header、設定值或輸入資料要依實際使用情境帶入。
// 回傳結果 / 副作用：可能回傳物件、集合、HTTP response、狀態碼，或造成資料寫入、log、外部服務呼叫等副作用。
type FrameworkConceptMap = {
  state: ['Vue ref', 'Svelte $state', 'Angular signal']
  derivedState: ['Vue computed', 'Svelte $derived', 'Angular computed']
  childInput: ['Vue props', 'Svelte props', 'Angular input']
  childOutput: ['Vue emit', 'Svelte callback/bindable', 'Angular output/model']
}
```


### 如果結果和預期不同

- 畫面沒有更新：先檢查狀態變數、props / events、模板語法與瀏覽器 console。
- 元件沒有渲染：確認元件是否正確 import、註冊、掛載，或 route / page 是否指到正確檔案。
- 建置或部署失敗：先看 terminal / GitHub Actions log 的第一個錯誤，不要只看最後一行。

### 做完後檢查

- 用瀏覽器實際操作一次本篇功能。
- 對照 Angular 中類似概念，寫下 Vue / Svelte 哪裡更像、哪裡不同。
- 確認沒有 console error，且重新整理頁面後仍能看到預期結果。

### 小練習

1. 把範例中的命名改成另一個真實業務場景。
2. 故意改錯一個 props、event 或 binding 名稱，觀察錯誤訊息。
3. 寫一句話比較 Angular 與本篇 Vue / Svelte 寫法的差異。

### 實務使用情境

- 做技術選型時，比較框架如何處理相同需求。
- 從一個框架轉職到另一個框架時，先找概念對照表。

### 不適合使用的情境

- 不應用玩具專案直接推論大型企業專案的全部維護成本，還要評估團隊、社群、測試、架構與工具鏈。

### 負面例子 / 錯誤用法

錯誤做法：看完系列後只記得哪個框架語法最短，直接宣稱它最好。

問題：語法長短不是唯一指標；可維護性、型別支援、團隊熟悉度、部署限制都重要。

修正方向：用具體需求評估框架，而不是只比較範例行數。

### Junior 常見誤解

- 「框架 A 比框架 B 短，所以一定更好」：簡潔是優點，但不是唯一決策因素。

### 一句話總結

框架會變，能跨框架轉移的是元件、狀態、資料流與非同步處理的基本功。

---

## 跨框架速查表

| 概念 | Vue 3 | Svelte 5 / SvelteKit | Angular 19/20 |
| --- | --- | --- | --- |
| 基本狀態 | `ref` / `reactive` | `$state` | `signal` |
| 衍生狀態 | `computed` | `$derived` | `computed` |
| 條件渲染 | `v-if` / `v-else` | `{#if}` | `@if` |
| 清單渲染 | `v-for` | `{#each}` | `@for` |
| 屬性綁定 | `:disabled` | `disabled={...}` | `[disabled]` |
| 事件 | `@click` / `@submit.prevent` | `onclick` / `onsubmit` | `(click)` / `(ngSubmit)` |
| 子元件輸入 | `defineProps` | `$props` | `input` / `@Input` |
| 子元件輸出 | `defineEmits` | callback props / `$bindable` | `output` / `@Output` |
| 內容投影 | slot | snippet / render | `ng-content` / `ng-template` |
| 動態元件 | `<component :is>` | dynamic component | `ngComponentOutlet` |
| 資料擷取封裝 | composable | load / store | service / `httpResource` |

## 最後整理

這個系列最值得帶走的不是「Vue、Svelte、Angular 誰比較好」，而是同一個產品需求在不同框架中會落到相似的工程問題：狀態放哪裡、資料怎麼傳、事件怎麼回報、非同步怎麼顯示、重複 UI 怎麼抽象。當你能用這些問題閱讀框架，學新框架就不再像從零開始。
