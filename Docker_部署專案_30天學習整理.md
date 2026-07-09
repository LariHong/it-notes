# Docker 部署專案 30 天學習整理

## 來源

- 使用者提供的系列頁：https://ithelp.ithome.com.tw/m/users/20151035/ironman/6311
- 原系列名稱：30 天學習 Docker 部署你的專案
- 原作者：krystal000
- 原系列：2023 iThome 鐵人賽 SideProject30
- 整理方式：依 30 篇公開文章的標題、系列順序、API 摘要與文章內容重新整理成教學筆記；不逐字重製原文。

## 這份筆記怎麼讀

這個系列不是只教 Docker 指令，而是把一個專案從本機容器化、推到 Docker Hub、放到 AWS EC2、再用 Traefik 與 Domain 做 HTTPS。讀的時候不要只背指令，要一直問三件事：這個指令改了哪個狀態、輸入從哪裡來、成功結果要在哪裡驗證。

建議讀法：先照 Day 1 到 Day 14 建立本機 Docker 基礎，再用 Day 15 到 Day 18 整理多容器與 image 發布，最後用 Day 19 到 Day 30 練部署、反向代理、HTTPS、Volume 與 image 優化。

## 2026 年閱讀時要注意的更新

- Docker Compose 現在通常使用 `docker compose` v2 指令，不再優先使用舊的 `docker-compose`。
- 部署到 EC2 時，要把 AWS security group、作業系統防火牆、container port mapping 三件事一起檢查。
- Traefik 與 Let's Encrypt 需要可解析的公開 Domain，單靠 EC2 IP 不足以穩定取得正式憑證。
- 正式環境不要把密碼、token、資料庫密碼直接寫進公開 repo 的 compose 檔，應改用 `.env`、secret manager 或部署平台變數。
- 本筆記的範例偏向學習用途，不宣稱是 production-ready；實務上還要補監控、備份、權限、更新策略與成本控管。

## 主線專案

### 專案最終會長成什麼

主線專案是一個可部署的 Web API：本機用 Dockerfile 打包，Docker Compose 串起 app、database、Traefik，最後部署到 EC2，透過 Domain 與 HTTPS 對外提供服務。

### 需要的檔案地圖

| 檔案 / 元件 | 負責的事情 |
| --- | --- |
| `src/DeployDemo.Api/` | 代表實際 Web 專案，可以是 ASP.NET Core、Rails、Node 或其他 Web app |
| `Dockerfile` | 定義如何把專案建成 image |
| `.dockerignore` | 排除不該進 image build context 的檔案 |
| `docker-compose.yml` | 定義 app、db、traefik、network、volume 的協作方式 |
| `.env` | 存放部署時注入的非公開設定；不要 commit 真實 secret |
| EC2 instance | 實際跑 container 的遠端主機 |
| Domain / DNS | 讓使用者用網址進入服務，也是 HTTPS 憑證申請基礎 |

### 30 天交付物地圖

| Day | 交付物 |
| --- | --- |
| 1 | 確認 Docker Desktop 可用，理解 container、image、registry 的基本角色 |
| 2 | 用 hello-world 驗證 Docker engine、pull image、create container、輸出結果這條鏈路 |
| 3 | 用 ps、images、pull、run、stop、rm、rmi 建立日常排查肌肉記憶 |
| 4 | 理解隔離、可重建、一次性與 host 資源共用的邊界 |
| 5 | 分清楚 create、start、run、stop、restart、rm，不把停止和刪除混在一起 |
| 6 | 讓容器用自訂 network 與服務名稱互通，不硬寫 localhost |
| 7 | 用 tag 表達版本與用途，避免 latest 造成部署不可追蹤 |
| 8 | 理解 registry 是遠端服務，repository 是 image 集合，push 前要登入與命名 |
| 9 | 理解 image layer、唯讀模板與 container 實例的關係 |
| 10 | 掌握 FROM、WORKDIR、COPY、RUN、CMD 的基本順序 |
| 11 | 補上 ENV、EXPOSE、ENTRYPOINT、USER 等執行階段設定 |
| 12 | 把實際專案打包成 image，讓新同事不必手動裝 runtime |
| 13 | 用 .dockerignore、cache-friendly COPY、multi-stage 前置概念縮短 build 時間 |
| 14 | 用 port mapping、env、logs 驗證 app container 真正提供服務 |
| 15 | 用 compose 同時啟動 app、db、reverse proxy 等多容器服務 |
| 16 | 理解 services、image/build、ports、environment、depends_on |
| 17 | 補上 networks、volumes、restart、healthcheck 與可維護設定 |
| 18 | 把本機 image 推到遠端，讓 EC2 或其他環境可以 pull |
| 19 | 建立 instance、security group、key pair，先把雲端主機準備好 |
| 20 | 在遠端主機安裝 Docker 並確認權限與服務狀態 |
| 21 | 在 instance pull/run image，用 security group 與 curl 驗證對外服務 |
| 22 | 把 compose 檔與 env 放上 EC2，用一條命令啟動整組服務 |
| 23 | 理解 reverse proxy、自動路由、entrypoints、providers、middleware |
| 24 | 用 Traefik router/service/entrypoint 讓服務從 HTTP 入口被代理 |
| 25 | 加入 Let's Encrypt resolver 與 TLS 設定，讓憑證自動申請 |
| 26 | 讓 DNS A record 指向 EC2 public IP，憑證申請前先確認 domain 解析 |
| 27 | 整合 domain、Traefik、Compose 與 app label，完成 HTTPS 對外服務 |
| 28 | 用 volume 保存資料庫與憑證資料，不讓 container 重建就清空狀態 |
| 29 | 用 build stage 與 runtime stage 減少 image 大小與攻擊面 |
| 30 | 用 container_name、service name、hostname 規劃可讀的部署命名 |

### 主線端到端流程

需求：把專案交給其他人並部署上線 -> Dockerfile 打包 image -> Compose 串多服務 -> Push image 到 Registry -> EC2 pull/run -> Traefik 代理 -> Domain 指向主機 -> HTTPS 驗證 -> Volume 保存狀態

### 主線做完後檢查

- 本機 `docker build` 與 `docker run` 可通過。
- `docker compose up -d` 後 app、db、traefik 狀態正常。
- EC2 上 `docker ps` 能看到服務，security group 開放必要 port。
- Domain 可解析，`https://<domain>` 回應正確。
- 重建 container 後，資料庫 volume 仍保存資料。

## 來源清單

| Day | 標題 | 連結 |
| --- | --- | --- |
| 1 | DAY 1 一起認識 Docker | https://ithelp.ithome.com.tw/articles/10319089 |
| 2 | DAY 2 Docker Hello World | https://ithelp.ithome.com.tw/articles/10320210 |
| 3 | DAY 3  Docker 基本指令 | https://ithelp.ithome.com.tw/articles/10321154 |
| 4 | DAY 4 Docker Container (容器) 的特點 | https://ithelp.ithome.com.tw/articles/10322269 |
| 5 | DAY 5 管理 Docker Container(容器) 的生命周期 | https://ithelp.ithome.com.tw/articles/10323001 |
| 6 | DAY 6 Docker Network(網路)容器與容器間的橋樑 | https://ithelp.ithome.com.tw/articles/10323931 |
| 7 | Day 7 - Docker Tags(標籤) 不只是標籤 | https://ithelp.ithome.com.tw/articles/10324739 |
| 8 | DAY 8 - Docker Registry (註冊表) 是倉庫的倉庫 | https://ithelp.ithome.com.tw/articles/10326050 |
| 9 | DAY 9 - Docker Image(映像) 用模型烤出一個個一樣的雞蛋糕 | https://ithelp.ithome.com.tw/articles/10326357 |
| 10 | DAY 10 - 拆解 Dockerfile 的關鍵字(上) | https://ithelp.ithome.com.tw/articles/10327632 |
| 11 | DAY 11 - 拆解 Dockerfile 的關鍵字(下) | https://ithelp.ithome.com.tw/articles/10327820 |
| 12 | DAY 12 - 撰寫自己專案的 Dockerfile | https://ithelp.ithome.com.tw/articles/10327821 |
| 13 | DAY 13 - 打磨我的 Dockerfile 製成 image | https://ithelp.ithome.com.tw/articles/10330055 |
| 14 | DAY 14 - 將我的 image 跑起來 | https://ithelp.ithome.com.tw/articles/10330972 |
| 15 | DAY 15 - Docker Compose 讓多個容器同時跑起來 | https://ithelp.ithome.com.tw/articles/10331547 |
| 16 | DAY 16 - 撰寫 Docker Compose 的 YAML 檔案(上) | https://ithelp.ithome.com.tw/articles/10332518 |
| 17 | DAY 17 - 撰寫 Docker Compose 的 YAML 檔案(下) | https://ithelp.ithome.com.tw/articles/10332966 |
| 18 | DAY 18 - 將我的 Image 推到 Docker Hub 儲存庫 | https://ithelp.ithome.com.tw/articles/10333562 |
| 19 | DAY 19 - 認識及建立 AWS EC2 Instance | https://ithelp.ithome.com.tw/articles/10334115 |
| 20 | DAY 20 - 連接到 EC2 instance 並下載 Docker | https://ithelp.ithome.com.tw/articles/10334594 |
| 21 | DAY 21 - 在 instance 上跑 Docker Image | https://ithelp.ithome.com.tw/articles/10335418 |
| 22 | DAY 22 - 在 EC2 instance 加上 docker compose 跑起來 | https://ithelp.ithome.com.tw/articles/10336096 |
| 23 | DAY 23 - 了解 Traefik 反向代理伺服器 | https://ithelp.ithome.com.tw/articles/10336768 |
| 24 | DAY 24 - 在我部署的專案使用 Traefik 取得 HTTPS 協定(一) | https://ithelp.ithome.com.tw/articles/10337284 |
| 25 | DAY 25 - 在我部署的專案使用 Traefik 取得 HTTPS 協定(二) | https://ithelp.ithome.com.tw/articles/10337821 |
| 26 | DAY 26 - 要取得協定得先有正確的 Domain | https://ithelp.ithome.com.tw/articles/10338280 |
| 27 | DAY 27 - 在我部署的專案使用 Traefik 取得 HTTPS 協定(三) | https://ithelp.ithome.com.tw/articles/10338739 |
| 28 | DAY 28 - 使用 Volumes 讓我的資料庫保存之前的資料 | https://ithelp.ithome.com.tw/articles/10339164 |
| 29 | DAY 29 - 將 Dockerfile 改成多階段建置 | https://ithelp.ithome.com.tw/articles/10339632 |
| 30 | DAY 30 - 將 docker-compose.yml 裡的 container 取名 | https://ithelp.ithome.com.tw/articles/10339949 |

## Day 1 - DAY 1 一起認識 Docker

來源：https://ithelp.ithome.com.tw/articles/10319089

### 這篇文章主要在講什麼

這篇聚焦在「Docker 的定位與安裝」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：因為工作上剛好需要將專案自動部署，所以接觸了 Docker ，所以就決定用文章來讓大家一起了解為何我需要用 Docker ，以及介紹一些基礎語法。整理成工作筆記時，可以把它看成主線專案的「確認 Docker Desktop 可用，理解 container、image、registry 的基本角色」。

原文可辨識的重點標題包含：Docker 是什麼？、Docker 想解決的問題及為何我需要使用 Docker、docker run、docker compose up、docker build。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Docker 的定位與安裝 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：確認 Docker Desktop 可用，理解 container、image、registry 的基本角色。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 因為工作上剛好需要將專案自動部署，所以接觸了 Docker ，所以就決定用文章來讓大家一起了解為何我需要用 Docker ，以及介紹一些基礎語法。
- Docker 是一個開源的容器化平台，他使用 Google 公司推出的 Go 語言實作，用於將應用程式、環境和相依套件打包成 輕量 、 可移植 的容器。
- 這些容器可以在任何地方運行，包括開發機器、測試環境和生產伺服器，確保應用程式在不同環境中的一致運行。
- 延伸整理：這一天的核心不是背 `docker version / docker info`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Docker 的定位與安裝」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Docker 的定位與安裝」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker version
docker info
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Docker 的定位與安裝」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Docker 的定位與安裝」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Docker 的定位與安裝」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Docker 的定位與安裝」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「確認 Docker Desktop 可用，理解 container、image、registry 的基本角色」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 1 的重點是：用「Docker 的定位與安裝」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 2 - DAY 2 Docker Hello World

來源：https://ithelp.ithome.com.tw/articles/10320210

### 這篇文章主要在講什麼

這篇聚焦在「Hello World 與第一個容器」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：昨天我們安裝了環境，但我們要如何確保電腦可以跑 Docker 了呢。整理成工作筆記時，可以把它看成主線專案的「用 hello-world 驗證 Docker engine、pull image、create container、輸出結果這條鏈路」。

原文可辨識的重點標題包含：Docker 基礎架構、Docker client(Docker 客戶端)、Docker daemon(Docker 守護進程)、Docker API、docker run hello-world。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Hello World 與第一個容器 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：用 hello-world 驗證 Docker engine、pull image、create container、輸出結果這條鏈路。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 昨天我們安裝了環境，但我們要如何確保電腦可以跑 Docker 了呢。
- 這時候就要來執行學習程式語言的起手式 Hello World ，在 Docker 中只要成功跑 Hello World 就是代表成功安裝。
- 但在這之前我們需要先知道 Docker 架構，以及學習 Docker 必用的三寶。
- 延伸整理：這一天的核心不是背 `docker run hello-world`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Hello World 與第一個容器」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Hello World 與第一個容器」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker run hello-world
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Hello World 與第一個容器」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Hello World 與第一個容器」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Hello World 與第一個容器」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Hello World 與第一個容器」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「用 hello-world 驗證 Docker engine、pull image、create container、輸出結果這條鏈路」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 2 的重點是：用「Hello World 與第一個容器」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 3 - DAY 3  Docker 基本指令

來源：https://ithelp.ithome.com.tw/articles/10321154

### 這篇文章主要在講什麼

這篇聚焦在「基本指令」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：昨天我們在 Docker 中成功跑 Hello World ，今天來了解一些 Docker 的基本指令 要知道 Docker 有哪些指令可以打的話，我們可以直接在終端機下 docker 這個指令，這時可以看到所有 docker 可以下的指令 其中我們可以看到三大分類 是用於設定 Docker 執行環境的全域設定。整理成工作筆記時，可以把它看成主線專案的「用 ps、images、pull、run、stop、rm、rmi 建立日常排查肌肉記憶」。

原文可辨識的重點標題包含：docker、Options（選項）：、docker --help、docker -v、docker --version。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。基本指令 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：用 ps、images、pull、run、stop、rm、rmi 建立日常排查肌肉記憶。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 昨天我們在 Docker 中成功跑 Hello World ，今天來了解一些 Docker 的基本指令 要知道 Docker 有哪些指令可以打的話，我們可以直接在終端機下 docker 這個指令，這時可以看到所有 docker 可以下的指令 其中我們可以看到三大分類 是用於設定 Docker 執行環境的全域設定。
- 例如： docker --help 可以看到 docker 列表 docker -v 及 docker --version 可以看到現在 docker 的版本 這些命令用於管理 Docker 容器、映像、網絡。
- 例如： docker container 命令是用來管理容器 ; docker image 命令是用來管理映像 ; docker network 命令是用來管理網絡。
- 延伸整理：這一天的核心不是背 `docker ps -a / docker images`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「基本指令」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「基本指令」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker ps -a
docker images
docker pull nginx:alpine
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「基本指令」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「基本指令」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「基本指令」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「基本指令」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「用 ps、images、pull、run、stop、rm、rmi 建立日常排查肌肉記憶」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 3 的重點是：用「基本指令」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 4 - DAY 4 Docker Container (容器) 的特點

來源：https://ithelp.ithome.com.tw/articles/10322269

### 這篇文章主要在講什麼

這篇聚焦在「容器特性」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：在我們使用 Docker Container(容器)時，一定要先知道他到底好在哪裡。整理成工作筆記時，可以把它看成主線專案的「理解隔離、可重建、一次性與 host 資源共用的邊界」。

原文可辨識的重點標題包含：Docker Container(容器)的特點、隔離性：、輕量級：、可移植性：、快速部署：。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。容器特性 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：理解隔離、可重建、一次性與 host 資源共用的邊界。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 在我們使用 Docker Container(容器)時，一定要先知道他到底好在哪裡。
- Docker Container(容器)是是一種輕量級、可移植的執行環境，能夠確保在不同的環境中具有一致的執行行為。
- 但如果講這樣大家都了解那我文章也不用寫了，謝謝明天見 Docker Container(容器)確保容器與容器之間幾乎不互相影響，所以不同 Container(容器)能夠在同一主機上一起執行，而互不干擾。
- 延伸整理：這一天的核心不是背 `docker run --rm alpine sh -c "cat /etc/os-release"`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「容器特性」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「容器特性」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker run --rm alpine sh -c "cat /etc/os-release"
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「容器特性」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「容器特性」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「容器特性」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「容器特性」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「理解隔離、可重建、一次性與 host 資源共用的邊界」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 4 的重點是：用「容器特性」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 5 - DAY 5 管理 Docker Container(容器) 的生命周期

來源：https://ithelp.ithome.com.tw/articles/10323001

### 這篇文章主要在講什麼

這篇聚焦在「容器生命週期」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：當我們想要執行一個 Container 容器時，會使用 docker run 這個指令，這個指令會從我本地的 image 或 Docker Hub 儲存庫中下載名為 ruby 的 image ，然後使用這個 image 建立一個新的 container 並執行它。整理成工作筆記時，可以把它看成主線專案的「分清楚 create、start、run、stop、restart、rm，不把停止和刪除混在一起」。

原文可辨識的重點標題包含：Docker Container 的啟動、開始、停止、刪除、啟動 Container、docker run、docker run ruby、ruby。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。容器生命週期 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：分清楚 create、start、run、stop、restart、rm，不把停止和刪除混在一起。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 當我們想要執行一個 Container 容器時，會使用 docker run 這個指令，這個指令會從我本地的 image 或 Docker Hub 儲存庫中下載名為 ruby 的 image ，然後使用這個 image 建立一個新的 container 並執行它。
- docker run ruby 以上面的指令為例，會先看我本地是否有名為 ruby 的 image ，若有就根據這個 image 生成 container 並執行它 ; 若無，就去 Docker Hub 儲存庫中下載名為 ruby 的 image ，然後建立新的 container 並執行它。
- 根據上面截圖的第一句可以看到 Unable to find image 'ruby:latest' locally 代表本地沒有找到 ruby:latest 的 image ，所以可以看到第二句 latest: Pulling from library/ruby ，代表從 Docker Hub 的 library/ruby 存儲庫中下載 ruby:latest 映像。
- 延伸整理：這一天的核心不是背 `docker create --name demo nginx:alpine / docker start demo`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「容器生命週期」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「容器生命週期」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker create --name demo nginx:alpine
docker start demo
docker stop demo
docker rm demo
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「容器生命週期」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「容器生命週期」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「容器生命週期」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「容器生命週期」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「分清楚 create、start、run、stop、restart、rm，不把停止和刪除混在一起」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 5 的重點是：用「容器生命週期」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 6 - DAY 6 Docker Network(網路)容器與容器間的橋樑

來源：https://ithelp.ithome.com.tw/articles/10323931

### 這篇文章主要在講什麼

這篇聚焦在「Docker network」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：前一章有說 Docker Container (容器)是有隔離性的，也就是互不影響，但這樣的話，我們應該要如何連結兩個容器呢。整理成工作筆記時，可以把它看成主線專案的「讓容器用自訂 network 與服務名稱互通，不硬寫 localhost」。

原文可辨識的重點標題包含：Bridge Network (橋接網絡)：、Host Network（主機網路）：、None Network（無網路）：、查看本機 docker 網路列表、docker network ls。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Docker network 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：讓容器用自訂 network 與服務名稱互通，不硬寫 localhost。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 前一章有說 Docker Container (容器)是有隔離性的，也就是互不影響，但這樣的話，我們應該要如何連結兩個容器呢。
- 這時候 Docker Network (網路)就出場啦。
- Docker Network (網路)是 Docker Container (容器)與 Container (容器)之間 ; 以及 Container (容器)與外部容器、網路或服務之間連接通訊的橋樑。
- 延伸整理：這一天的核心不是背 `docker network create app-net / docker run -d --name web --network app-net nginx:alpine`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Docker network」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Docker network」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker network create app-net
docker run -d --name web --network app-net nginx:alpine
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Docker network」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Docker network」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Docker network」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Docker network」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「讓容器用自訂 network 與服務名稱互通，不硬寫 localhost」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 6 的重點是：用「Docker network」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 7 - Day 7 - Docker Tags(標籤) 不只是標籤

來源：https://ithelp.ithome.com.tw/articles/10324739

### 這篇文章主要在講什麼

這篇聚焦在「Image tag」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：在 Docker Container(容器) 篇，我們曾短暫的介紹過 Docker Tags ，但那只是最表面的，但其實 Docker Tags(標籤) 它不只是標籤，今天我們就來深入了解一下 Docker Tags(標籤) 還有哪些功用吧。整理成工作筆記時，可以把它看成主線專案的「用 tag 表達版本與用途，避免 latest 造成部署不可追蹤」。

原文可辨識的重點標題包含：docker run ruby、ruby:latest、建立自己的 Tags(標籤)、docker images、docker tag <本機現有映像 id_or_repository>:<本機現有標籤> <新映像 repository>:<新標籤>。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Image tag 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：用 tag 表達版本與用途，避免 latest 造成部署不可追蹤。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 在 Docker Container(容器) 篇，我們曾短暫的介紹過 Docker Tags ，但那只是最表面的，但其實 Docker Tags(標籤) 它不只是標籤，今天我們就來深入了解一下 Docker Tags(標籤) 還有哪些功用吧。
- 先來複習一下之前說到的當我們想要啟動一個 Container(容器) 時，打了 docker run ruby 時，因為本地沒有叫做 ruby 的 Image(映像) ，所以他自動去遠端儲存庫找 ruby:latest 這個 ruby Image(映像) ，以及使用預設的 latest Tags(標籤) ，所以我們了解到我們可以在 Docker Hub 看到及選擇 pull 或 run 不同 Tags(標籤)的 ruby Image(映像)，但這些其實都是在使用別人建好的 Image(映像) 或是 Tags(標籤) ，其實我們也可以建立屬於自己的 Tags(標籤) 呦。
- 首先我們可以先使用 docker images 看一下自己現在本機所有的 Image(映像) 這邊我們可以看到有之前我們 run 時，從公共儲存庫 pull 下來的 ruby Image(映像) ，這時我們可以使用以下語法來建立一個有新標籤的 Image(映像) docker tag <本機現有映像 id_or_repository>:<本機現有標籤> <新映像 repository>:<新標籤> 這個語法是為本機原本的 ruby Image(映像)，建立一個新的 Image(映像) ，而這個 Image(映像)是新的標籤 v1 ，所以當你看 docker images 時，會看到兩個 ruby Repository(倉庫)，但是是不同 Tags(標籤) - latest 跟 v1。
- 延伸整理：這一天的核心不是背 `docker tag app:dev yourname/app:2026-07-09`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Image tag」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Image tag」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker tag app:dev yourname/app:2026-07-09
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Image tag」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Image tag」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Image tag」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Image tag」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「用 tag 表達版本與用途，避免 latest 造成部署不可追蹤」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 7 的重點是：用「Image tag」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 8 - DAY 8 - Docker Registry (註冊表) 是倉庫的倉庫

來源：https://ithelp.ithome.com.tw/articles/10326050

### 這篇文章主要在講什麼

這篇聚焦在「Registry / Repository」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：在前面介紹 docker 三寶時，有介紹到 Docker Repository（儲存庫），可以把它想像成存放 Docker Image(映像) 的倉庫，那今天要介紹的 Docker Registry (註冊表) 就可以把它想像成倉庫的倉庫，那究竟倉庫的倉庫是什麼意思呢。整理成工作筆記時，可以把它看成主線專案的「理解 registry 是遠端服務，repository 是 image 集合，push 前要登入與命名」。

原文可辨識的重點標題包含：先複習 Docker Repository（儲存庫）、docker images、myapp、ruby、myapp。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Registry / Repository 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：理解 registry 是遠端服務，repository 是 image 集合，push 前要登入與命名。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 在前面介紹 docker 三寶時，有介紹到 Docker Repository（儲存庫），可以把它想像成存放 Docker Image(映像) 的倉庫，那今天要介紹的 Docker Registry (註冊表) 就可以把它想像成倉庫的倉庫，那究竟倉庫的倉庫是什麼意思呢。
- 我們可以把多個不同版本和標籤的 Docker Image(映像)，放在 Docker Repository（儲存庫）這個倉庫保存著，每個 Docker 倉庫都只能有一個名稱，但可能有一個或多個 Tag (標籤) ，用來辨別 Image(映像) ，這也就是為什麼我們前一章使用 docker images 指令時，出現的是這些資訊 所以像是 myapp 以及 ruby 都是不同 Docker Repository（儲存庫），但是 myapp 這個 Docker Repository（儲存庫）可以有如 latest 與 v1 一個或多個 Tag (標籤)，所以我們可以用 myapp:v1 來辨別要找哪個 Image(映像)。
- 直接講結論就是，一個 Docker Registry (註冊表)可以包含多個 Docker Repository（儲存庫），每個 Docker Repository（儲存庫）可以包含多個不同版本的 Docker Image(映像)。
- 延伸整理：這一天的核心不是背 `docker login / docker push yourname/app:2026-07-09`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Registry / Repository」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Registry / Repository」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker login
docker push yourname/app:2026-07-09
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Registry / Repository」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Registry / Repository」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Registry / Repository」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Registry / Repository」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「理解 registry 是遠端服務，repository 是 image 集合，push 前要登入與命名」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 8 的重點是：用「Registry / Repository」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 9 - DAY 9 - Docker Image(映像) 用模型烤出一個個一樣的雞蛋糕

來源：https://ithelp.ithome.com.tw/articles/10326357

### 這篇文章主要在講什麼

這篇聚焦在「Image」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：Docker Image(映像) 的特性是輕量且獨立，其中包含了運行應用程式所需的代碼、工具、資料庫和設置等等。整理成工作筆記時，可以把它看成主線專案的「理解 image layer、唯讀模板與 container 實例的關係」。

原文可辨識的重點標題包含：Image(映像) 的特性、不可變性：、分層結構：、Docker Image(映像) 的常用語法、查看本地的所有 Image(映像)。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Image 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：理解 image layer、唯讀模板與 container 實例的關係。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- Docker Image(映像) 的特性是輕量且獨立，其中包含了運行應用程式所需的代碼、工具、資料庫和設置等等。
- Docker Image(映像) 不但方便移植，且封裝應用程式，還有他們互相依賴的關係，我們可以在不同的環境中部署和運行 Docker Image(映像) ，對我來說最大的優點是，不用擔心環境差異或配置的問題。
- Docker Image(映像) 是不可變的，一旦構建完成，就不能更改。
- 延伸整理：這一天的核心不是背 `docker history nginx:alpine`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Image」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Image」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker history nginx:alpine
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Image」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Image」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Image」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Image」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「理解 image layer、唯讀模板與 container 實例的關係」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 9 的重點是：用「Image」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 10 - DAY 10 - 拆解 Dockerfile 的關鍵字(上)

來源：https://ithelp.ithome.com.tw/articles/10327632

### 這篇文章主要在講什麼

這篇聚焦在「Dockerfile 上」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：回到我前幾天介紹的 Docker Image(映像) 我們把它想像成烤雞蛋糕模型，那這個 Dockerfile 我們就可以把它想像成這個模型的說明書，這本說明書包含了一系列指令，告訴 Docker 如何建立一个映像。整理成工作筆記時，可以把它看成主線專案的「掌握 FROM、WORKDIR、COPY、RUN、CMD 的基本順序」。

原文可辨識的重點標題包含：Dockerfile 是什麼？、Dockerfile 裡面的關鍵字、FROM、FROM ruby:3.2.2、AS。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Dockerfile 上 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：掌握 FROM、WORKDIR、COPY、RUN、CMD 的基本順序。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 回到我前幾天介紹的 Docker Image(映像) 我們把它想像成烤雞蛋糕模型，那這個 Dockerfile 我們就可以把它想像成這個模型的說明書，這本說明書包含了一系列指令，告訴 Docker 如何建立一个映像。
- 當我們要 run docker 時，他會根據現在專案的根目錄，找尋 Dockerfile ，並且根據 Dockerfile 裡面一步一步的說明，來安裝好相關套件資料庫等環境，然後製作成一個 Image(映像)。
- 當然如果你的 Dockerfile 沒有在根目錄也是可以的，只要指定對路徑，能找到 Dockerfile 就可以根據 Dockerfile 來製作 Image(映像)。
- 延伸整理：這一天的核心不是背 `docker build -t deploy-demo:day10 .`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Dockerfile 上」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Dockerfile 上」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker build -t deploy-demo:day10 .
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Dockerfile 上」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Dockerfile 上」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Dockerfile 上」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Dockerfile 上」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「掌握 FROM、WORKDIR、COPY、RUN、CMD 的基本順序」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 10 的重點是：用「Dockerfile 上」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 11 - DAY 11 - 拆解 Dockerfile 的關鍵字(下)

來源：https://ithelp.ithome.com.tw/articles/10327820

### 這篇文章主要在講什麼

這篇聚焦在「Dockerfile 下」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：昨天了解上半部的 Dockerfile 關鍵字，今天繼續來把 Dockerfile 的關鍵字學完吧～ 在容器內部設定的環境變數 ENV DB_HOST=postgresql 設定一個環境變數叫做 DB_HOST ， value 是 postgresql 在 Dockerfile 中使用的參數，與 ENV 作用很類似，但是作用域不一樣。整理成工作筆記時，可以把它看成主線專案的「補上 ENV、EXPOSE、ENTRYPOINT、USER 等執行階段設定」。

原文可辨識的重點標題包含：ENV、ENV DB_HOST=postgresql、DB_HOST、postgresql、ARG。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Dockerfile 下 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：補上 ENV、EXPOSE、ENTRYPOINT、USER 等執行階段設定。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 昨天了解上半部的 Dockerfile 關鍵字，今天繼續來把 Dockerfile 的關鍵字學完吧～ 在容器內部設定的環境變數 ENV DB_HOST=postgresql 設定一個環境變數叫做 DB_HOST ， value 是 postgresql 在 Dockerfile 中使用的參數，與 ENV 作用很類似，但是作用域不一樣。
- ARG 設定的環境變數只有在 Dockerfile 內，及 docker build 的過程中有效，建立成 Image(映像) 內，就沒有這個環境變數 ; 但 ENV 則是容器啟動時，也可以使用的變數。
- ARG <参数名>[=<默认值>] ARG DB_HOST=postgresql 是設定這個容器內部現在的資料夾 WORKDIR /app 代表 Docker 會在容器裡面建立一個名為 /app 的資料夾，且現在就在這個資料夾裡，我自己是把它想像成這個容器的根目錄。
- 延伸整理：這一天的核心不是背 `docker run --rm -e ASPNETCORE_ENVIRONMENT=Development deploy-demo:day11`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Dockerfile 下」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Dockerfile 下」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker run --rm -e ASPNETCORE_ENVIRONMENT=Development deploy-demo:day11
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Dockerfile 下」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Dockerfile 下」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Dockerfile 下」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Dockerfile 下」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「補上 ENV、EXPOSE、ENTRYPOINT、USER 等執行階段設定」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 11 的重點是：用「Dockerfile 下」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 12 - DAY 12 - 撰寫自己專案的 Dockerfile

來源：https://ithelp.ithome.com.tw/articles/10327821

### 這篇文章主要在講什麼

這篇聚焦在「專案 Dockerfile」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：前兩天我們了解了 Dockerfile 所有的關鍵字，就是為了今天我們實際寫的時候可以有些概念，會比較好上手一點點。整理成工作筆記時，可以把它看成主線專案的「把實際專案打包成 image，讓新同事不必手動裝 runtime」。

原文可辨識的重點標題包含：新增 Dockerfile 檔案、Dockerfile、docker build -f MyDockerfile .、-f、file。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。專案 Dockerfile 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：把實際專案打包成 image，讓新同事不必手動裝 runtime。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 前兩天我們了解了 Dockerfile 所有的關鍵字，就是為了今天我們實際寫的時候可以有些概念，會比較好上手一點點。
- 不確定大家手邊是否都有專案，所以我這邊有準備一個示範的 rails 專案，有需要的人可以從 這邊 將專案給 clone 下來，在預設的 main 分支就可以一步一步跟著做了。
- 順利打開專案後（以下範例我都是使用 Visual Studio Code 這個編輯器），首先我們需要先在專案的 根目錄 新增一個名為 Dockerfile 的檔案 📍Dockerfile 是在 Docker 中預設的 Docker 映像建置檔的預設名字，若你想要取自己想要的名字也可以，但就是在建置 image 的時候，需要說明清楚用哪個文件來建置 docker build -f MyDockerfile . 上面 -f 是 file 的縮寫，正常取名為 Dockerfile 就不用特別寫，但如果需要自定義名稱時，如： MyDockerfile ，那就要在建置 image 時，使用 -f 標清楚 另外前幾天也有提到，我們預設是將 Dockerfile 放在根目錄，但如果我希望把 Dockerfile 放在專案路徑 docker/Dockerfile 時，就一樣需要使用 -f 標明清楚 docker build -f docker/Dockerfile . 那我們這邊一樣根據預設在根目錄建立一個 Dockerfile 檔案就好，接著我們要來寫 FROM ，所以首先要先來選擇這個專案的基礎鏡像版本，首先先確認一下這個專案使用的 ruby 版本 我可以在 Visual Studio Code 的終端機上使用以下語法，來查看專案的 ruby 版本 ruby -v 2. 若是 clone 的專案，那可以參考那個專案的 README.md ，正常都會在專案寫下使用的版本 或是可以在專案裡找版本或配置的檔案，在 rails 專案就可以看 Gemfile 檔案 上面這麼多種方法我們都可以得知是使用 ruby 3.2.2 ，那這時我們就要去 Docker Hub ruby 來找尋是否有符合的 image 看到後我又暈了，這麼多不同的 tag 都是 3.2.2 我到底要選用哪一個＠＠ Docker Hub ruby 往下滑，有介紹不同 Variants 的差別是什麼可以幫助大家選擇 This is the defacto image. If you are unsure about what your needs are, you probably want to use this one. It is designed to be used both as a throw away container (mount your source code and start the container to start your app), as well as the base to build other images off of. 根據 ruby:<version> 上面的說明，「如果你不確定自己的需求是什麼，你可能想使用這個」，沒錯我自己也是覺得當不知道要選擇哪個時，選擇一個最標準的 image 準沒錯，所以我在剛剛眾多的 tags 找到了我的超人。
- 延伸整理：這一天的核心不是背 `docker build -t deploy-demo:day12 .`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「專案 Dockerfile」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「專案 Dockerfile」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker build -t deploy-demo:day12 .
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「專案 Dockerfile」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「專案 Dockerfile」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「專案 Dockerfile」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「專案 Dockerfile」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「把實際專案打包成 image，讓新同事不必手動裝 runtime」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 12 的重點是：用「專案 Dockerfile」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 13 - DAY 13 - 打磨我的 Dockerfile 製成 image

來源：https://ithelp.ithome.com.tw/articles/10330055

### 這篇文章主要在講什麼

這篇聚焦在「打磨 Dockerfile」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：昨天我們將 Dockerfile 撰寫大概有個樣子了，今天就馬上來根據 Dockerfile 來建立自己的 image 吧。整理成工作筆記時，可以把它看成主線專案的「用 .dockerignore、cache-friendly COPY、multi-stage 前置概念縮短 build 時間」。

原文可辨識的重點標題包含：根據 Dockerfile 建立自己的 image、docker build -t my-ruby:1.0 .、bundle install、1.、RUN bundle config。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。打磨 Dockerfile 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：用 .dockerignore、cache-friendly COPY、multi-stage 前置概念縮短 build 時間。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 昨天我們將 Dockerfile 撰寫大概有個樣子了，今天就馬上來根據 Dockerfile 來建立自己的 image 吧。
- （ 就是一個瘋狂除錯的步驟🥹 ） 接著立馬試試能否成功建立自己的 image docker build -t my-ruby:1.0 . 如圖示可以看到在 bundle install 時失敗了。
- 錯誤訊息還不只一兩個😅 Bundler 2.4.10 is running, but your lockfile was generated with 2.4.18. Installing Bundler 2.4.18 and restarting using that version. 看起來是 Bundler 版本不一致所導致的。
- 延伸整理：這一天的核心不是背 `docker build --no-cache -t deploy-demo:day13 .`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「打磨 Dockerfile」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「打磨 Dockerfile」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker build --no-cache -t deploy-demo:day13 .
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「打磨 Dockerfile」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「打磨 Dockerfile」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「打磨 Dockerfile」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「打磨 Dockerfile」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「用 .dockerignore、cache-friendly COPY、multi-stage 前置概念縮短 build 時間」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 13 的重點是：用「打磨 Dockerfile」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 14 - DAY 14 - 將我的 image 跑起來

來源：https://ithelp.ithome.com.tw/articles/10330972

### 這篇文章主要在講什麼

這篇聚焦在「跑起自己的 image」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：將建立好的 image 製成 container 前面使用 docker build -t my-ruby:1.0 . --load 只是單純建立好模型，但是當你希望他實際上可以運轉時，還是要使用 docker run 的指令將 image 啟動一個新的 container docker run my-ruby:1.0 得到以下輸出 tzinfo-data is not present. Please add gem 'tzinfo-data' to your Gemfile and run bundle install (TZInfo::DataSourceNotFound) 現在已經對錯誤麻木了，我把錯誤訊息貼給谷歌大神，再加上 docker 這個關鍵字， 第一個搜尋結果 答案就出來了。整理成工作筆記時，可以把它看成主線專案的「用 port mapping、env、logs 驗證 app container 真正提供服務」。

原文可辨識的重點標題包含：docker build -t my-ruby:1.0 . --load、docker run、docker run my-ruby:1.0、tzdata、Listening on http://0.0.0.0:3000。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。跑起自己的 image 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：用 port mapping、env、logs 驗證 app container 真正提供服務。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 將建立好的 image 製成 container 前面使用 docker build -t my-ruby:1.0 . --load 只是單純建立好模型，但是當你希望他實際上可以運轉時，還是要使用 docker run 的指令將 image 啟動一個新的 container docker run my-ruby:1.0 得到以下輸出 tzinfo-data is not present. Please add gem 'tzinfo-data' to your Gemfile and run bundle install (TZInfo::DataSourceNotFound) 現在已經對錯誤麻木了，我把錯誤訊息貼給谷歌大神，再加上 docker 這個關鍵字， 第一個搜尋結果 答案就出來了。
- 原來我前面因為不認識所拿掉的 tzdata 是負責處理時區的套件，而這也是相依套件，所以我們再把它加回 Dockerfile ，重新 buile 跟 run 試試看 FROM ruby:3.2.2-alpine LABEL maintainer="Krystal <krystal@5xcampus.com>" RUN apk add \ build-base \ postgresql-dev \ tzdata WORKDIR /app COPY Gemfile Gemfile.lock ./ RUN bundle install COPY . . EXPOSE 3000 CMD ["rails", "server", "-b", "0.0.0.0"] 雖然這個過程可能很繁瑣，但是我們在建置 Dockerfile 時，真的實際就是這樣，一開始不知道有哪些相異套件要用 RUN 裝進容器裡，都是一次次的錯誤，一個個加上去的。
- 他甚至跟我說 Listening on http://0.0.0.0:3000 我還不手刀打開瀏覽器，輸入網址 http://0.0.0.0:3000 關鍵字搜尋 again docker run image localhost not found ，看到這篇 stackoverflow ，決定試試 # docker run -p <host-port>:<container-port> imageName docker run -p 3000:3000 my-ruby:1.0 因為 rails 是 3000 port，所以我們要提醒他要監聽的 port 因為我們要重新 run 一個相同的 port，又或是說現在這個 container 我們不用了，所以要先停止或退出容器，可以使用以下方法退出 Ctrl-C docker stop <container_id 或 container_name> 順利退出後會有 - Goodbye! Exiting 的字樣，這時再重新使用 docker run -p 3000:3000 my-ruby:1.0 重新開起新的容器，並告訴他要監聽的 port 是 3000 port，再次順利開起，打開瀏覽器，輸入網址 http://0.0.0.0:3000 至少，有錯誤訊息，而且這個錯誤看起來像是 rails 專案會看到的紅畫面，裡面說的內容感覺跟資料庫有關，算是一個大進步的錯誤訊息，至少代表我們成功用 docker 開啟 rails 專案了。
- 延伸整理：這一天的核心不是背 `docker run --rm -p 8080:8080 deploy-demo:day14`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「跑起自己的 image」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「跑起自己的 image」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker run --rm -p 8080:8080 deploy-demo:day14
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「跑起自己的 image」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「跑起自己的 image」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「跑起自己的 image」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「跑起自己的 image」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「用 port mapping、env、logs 驗證 app container 真正提供服務」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 14 的重點是：用「跑起自己的 image」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 15 - DAY 15 - Docker Compose 讓多個容器同時跑起來

來源：https://ithelp.ithome.com.tw/articles/10331547

### 這篇文章主要在講什麼

這篇聚焦在「Docker Compose」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：前三天不小心陷入 Dockerfile 的除錯地獄中🤢，甚至最後留下的畫面都還是紅畫面，像極了在一片黑的洞穴中，找不到任何一絲光線的出口，剛開始學 docker 很容易因為遇到這種狀況所以就放棄了。整理成工作筆記時，可以把它看成主線專案的「用 compose 同時啟動 app、db、reverse proxy 等多容器服務」。

原文可辨識的重點標題包含：Docker Compose 是什麼？、FROM ruby:3.2.2、為何我要用 Docker Compose ？、優點、Docker Compose 常見指令。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Docker Compose 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：用 compose 同時啟動 app、db、reverse proxy 等多容器服務。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 前三天不小心陷入 Dockerfile 的除錯地獄中🤢，甚至最後留下的畫面都還是紅畫面，像極了在一片黑的洞穴中，找不到任何一絲光線的出口，剛開始學 docker 很容易因為遇到這種狀況所以就放棄了。
- 所以今天我們先轉換一下口味，脫離除錯地獄，轉換心情來學習 Docker Compose 是什麼，當然這也是我之後要拿來解決昨天紅畫面錯誤訊息的解決方案。
- Docker Compose 是一個用於定義和執行多容器 Docker 應用程式的工具。
- 延伸整理：這一天的核心不是背 `docker compose up -d`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Docker Compose」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Docker Compose」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker compose up -d
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Docker Compose」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Docker Compose」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Docker Compose」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Docker Compose」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「用 compose 同時啟動 app、db、reverse proxy 等多容器服務」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 15 的重點是：用「Docker Compose」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 16 - DAY 16 - 撰寫 Docker Compose 的 YAML 檔案(上)

來源：https://ithelp.ithome.com.tw/articles/10332518

### 這篇文章主要在講什麼

這篇聚焦在「Compose YAML 上」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：一直說 YAML 檔案，但 YAML 到底是什麼。整理成工作筆記時，可以把它看成主線專案的「理解 services、image/build、ports、environment、depends_on」。

原文可辨識的重點標題包含：新增 Docker Compose YAML 檔案、docker-compose.yml、docker-compose.yaml、.yml、.yaml。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Compose YAML 上 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：理解 services、image/build、ports、environment、depends_on。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 一直說 YAML 檔案，但 YAML 到底是什麼。
- 📍 YAML 輕量且易於撰寫的檔案，主要可以將資料轉換成特定格式，常用於設定檔和配置檔的編寫。
- 跟 Dockerfile 一樣，我們可以新增一個 docker-compose.yml 或 docker-compose.yaml 檔案在根目錄，原因不外乎就是因為 .yml 或是 .yaml 都可以被識別為是 YAML 檔，那我秉持一貫懶惰的個性，就推薦直接用 .yml 為結尾。
- 延伸整理：這一天的核心不是背 `docker compose config`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Compose YAML 上」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Compose YAML 上」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker compose config
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Compose YAML 上」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Compose YAML 上」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Compose YAML 上」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Compose YAML 上」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「理解 services、image/build、ports、environment、depends_on」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 16 的重點是：用「Compose YAML 上」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 17 - DAY 17 - 撰寫 Docker Compose 的 YAML 檔案(下)

來源：https://ithelp.ithome.com.tw/articles/10332966

### 這篇文章主要在講什麼

這篇聚焦在「Compose YAML 下」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：昨天將 app 服務 docker-compose.yml 的部分撰寫好了，今天就來寫資料庫服務的部分 跟昨天一樣取一個資料庫在 compose 裡的唯一名稱 db: 為了簡明扼要又好讀，就直接叫做 db 昨天我們使用 build: context: . 是因為我們希望可以根據本地的 Dockerfile 來建置容器 ; 但今天我們的資料庫就是因為沒有在 Dockerfile 建立出來的容器裡，所以我們應該要找一個符合我安裝的 PostgreSQL 版本的 image 寫成： image: <我要引用的 image> 確認 PostgreSQL 版本 首先若是本機已經安裝 PostgreSQL 可以在終端機下 psql 使用 psql 指令連接到您的 PostgreSQL 資料庫，可以看到如下 這邊可以看到後面的 14.5 就是 PostgreSQL 服務的版本，但若是你還是想再確認一次，或是知道更多詳細訊息，也可以在這邊再下 SELECT version(); 如圖我們就可以知道有關版本的更多詳細訊息。整理成工作筆記時，可以把它看成主線專案的「補上 networks、volumes、restart、healthcheck 與可維護設定」。

原文可辨識的重點標題包含：將資料庫服務定義好、定義 PostgreSQL 服務名稱、db:、db、根據什麼建立容器？。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Compose YAML 下 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：補上 networks、volumes、restart、healthcheck 與可維護設定。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 昨天將 app 服務 docker-compose.yml 的部分撰寫好了，今天就來寫資料庫服務的部分 跟昨天一樣取一個資料庫在 compose 裡的唯一名稱 db: 為了簡明扼要又好讀，就直接叫做 db 昨天我們使用 build: context: . 是因為我們希望可以根據本地的 Dockerfile 來建置容器 ; 但今天我們的資料庫就是因為沒有在 Dockerfile 建立出來的容器裡，所以我們應該要找一個符合我安裝的 PostgreSQL 版本的 image 寫成： image: <我要引用的 image> 確認 PostgreSQL 版本 首先若是本機已經安裝 PostgreSQL 可以在終端機下 psql 使用 psql 指令連接到您的 PostgreSQL 資料庫，可以看到如下 這邊可以看到後面的 14.5 就是 PostgreSQL 服務的版本，但若是你還是想再確認一次，或是知道更多詳細訊息，也可以在這邊再下 SELECT version(); 如圖我們就可以知道有關版本的更多詳細訊息。
- 在 Docker Hub 找適合的 image 要找公開包好的 image ，第一時間想到 Docker Hub，我在 Docker Hub 裡搜尋 postgres 找尋版本 14 開頭的 tag 因為沒有完全符合 14.5 的 tag ，所以我就選擇 14-alpine ，這時 docker-compose.yml 可以寫成： image: postgres:14-alpine 剛剛找好根據哪個 image ，然後呢。
- 我們可以順勢參考頁面 postgres 這邊寫到需要設置環境變數，我們就加上去 environment: POSTGRES_PASSWORD: password 倒是有一個 restart: always 好像是先前沒有看過的 如其名就是重新開啟，再解釋清楚一點就是容器在什麼狀況下需要重新 restart ，通常用於確保容器在故障或異常情況下能夠恢復正常運行。
- 延伸整理：這一天的核心不是背 `docker compose ps`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Compose YAML 下」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Compose YAML 下」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker compose ps
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Compose YAML 下」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Compose YAML 下」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Compose YAML 下」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Compose YAML 下」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「補上 networks、volumes、restart、healthcheck 與可維護設定」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 17 的重點是：用「Compose YAML 下」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 18 - DAY 18 - 將我的 Image 推到 Docker Hub 儲存庫

來源：https://ithelp.ithome.com.tw/articles/10333562

### 這篇文章主要在講什麼

這篇聚焦在「Push 到 Docker Hub」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：為了要做到可以部署，我們需要先將我們專案包好的 image 推到 Docker Hub 的私有儲存庫，供之後可以來這邊拿取並啟動容器。整理成工作筆記時，可以把它看成主線專案的「把本機 image 推到遠端，讓 EC2 或其他環境可以 pull」。

原文可辨識的重點標題包含：註冊/登入 Docker Hub、建立私有儲存庫、Create repository、create、docker_test。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Push 到 Docker Hub 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：把本機 image 推到遠端，讓 EC2 或其他環境可以 pull。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 為了要做到可以部署，我們需要先將我們專案包好的 image 推到 Docker Hub 的私有儲存庫，供之後可以來這邊拿取並啟動容器。
- 翻成白話大概就是我要先把我的東西先放在倉庫，這樣之後只要有倉庫鑰匙的人，都可以去倉庫取貨。
- 搜尋 Docker Hub ，可以點選右上角的註冊或登入 若是跟我一樣已經有登入過，記住帳號的人，我們可以看到圖片中黃色框，他很貼心的顯示卡片可以讓我快速登入 登入後便會看到我所有的儲存庫，如下圖可以看到有兩條白白的，代表我有兩個儲存庫了，但因為有些資料不方便透露，所以我這邊先填白色，正常來說會有儲存庫的名稱等相關資料。
- 延伸整理：這一天的核心不是背 `docker tag deploy-demo:prod yourname/deploy-demo:prod / docker push yourname/deploy-demo:prod`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Push 到 Docker Hub」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Push 到 Docker Hub」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker tag deploy-demo:prod yourname/deploy-demo:prod
docker push yourname/deploy-demo:prod
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Push 到 Docker Hub」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Push 到 Docker Hub」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Push 到 Docker Hub」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Push 到 Docker Hub」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「把本機 image 推到遠端，讓 EC2 或其他環境可以 pull」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 18 的重點是：用「Push 到 Docker Hub」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 19 - DAY 19 - 認識及建立 AWS EC2 Instance

來源：https://ithelp.ithome.com.tw/articles/10334115

### 這篇文章主要在講什麼

這篇聚焦在「AWS EC2」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：AWS EC2 全名 Amazon Elastic Compute Cloud ，根據 官方文件 ，可以看到介紹 AWS EC2 提供最廣泛、最深入的運算平台，擁有超過 700 個執行個體，可選擇最新處理器、儲存、聯網、作業系統和購買模型，以協助您最有效地滿足工作負載需求。整理成工作筆記時，可以把它看成主線專案的「建立 instance、security group、key pair，先把雲端主機準備好」。

原文可辨識的重點標題包含：AWS EC2 是什麼？、AWS EC2 的優點、彈性伸縮：、多種執行個體類型：、作業系統靈活：。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。AWS EC2 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：建立 instance、security group、key pair，先把雲端主機準備好。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- AWS EC2 全名 Amazon Elastic Compute Cloud ，根據 官方文件 ，可以看到介紹 AWS EC2 提供最廣泛、最深入的運算平台，擁有超過 700 個執行個體，可選擇最新處理器、儲存、聯網、作業系統和購買模型，以協助您最有效地滿足工作負載需求。
- 我們是第一間支援 Intel、AMD 和 Arm 處理器的主要雲端供應商，提供唯一具有隨需 EC2 Mac 執行個體的雲端，以及唯一具有 400 Gbps 以太網路聯網的雲端。
- 我們為機器學習訓練提供了最佳價格效能，同時也是每個雲端推論執行個體的最低成本。
- 延伸整理：這一天的核心不是背 `ssh -i key.pem ec2-user@<EC2_PUBLIC_IP>`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「AWS EC2」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「AWS EC2」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
ssh -i key.pem ec2-user@<EC2_PUBLIC_IP>
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「AWS EC2」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「AWS EC2」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「AWS EC2」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「AWS EC2」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「建立 instance、security group、key pair，先把雲端主機準備好」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 19 的重點是：用「AWS EC2」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 20 - DAY 20 - 連接到 EC2 instance 並下載 Docker

來源：https://ithelp.ithome.com.tw/articles/10334594

### 這篇文章主要在講什麼

這篇聚焦在「EC2 安裝 Docker」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：先前我們可以使用 docker compose up ，是因為他可以根據我本機的 Dockerfile 製成一顆 Image ，並使用 compose 啟動 app 與 db 兩個服務，我現在希望可以將這個我在本機運作的流程，搬到 EC2 (遠端) 來實作，如此一來，任何人只要有網址或 ip 地址，都可以透過在瀏覽器搜尋看到我的專案，也就是達成部署的效果。整理成工作筆記時，可以把它看成主線專案的「在遠端主機安裝 Docker 並確認權限與服務狀態」。

原文可辨識的重點標題包含：連接到 AWS EC2、1. EC2 Instance Connect、2. SSH client、krystal.pem、krystal.pem。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。EC2 安裝 Docker 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：在遠端主機安裝 Docker 並確認權限與服務狀態。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 先前我們可以使用 docker compose up ，是因為他可以根據我本機的 Dockerfile 製成一顆 Image ，並使用 compose 啟動 app 與 db 兩個服務，我現在希望可以將這個我在本機運作的流程，搬到 EC2 (遠端) 來實作，如此一來，任何人只要有網址或 ip 地址，都可以透過在瀏覽器搜尋看到我的專案，也就是達成部署的效果。
- 在這之前我要在 EC2 裡面跑 Docker，所以我需要先連接到 EC2 以及 在 EC2 instance 裡下載 Docker。
- 🤯 其實說白話就是，建立我本機跟 AWS EC2 的網路連接。
- 延伸整理：這一天的核心不是背 `docker version / systemctl status docker`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「EC2 安裝 Docker」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「EC2 安裝 Docker」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker version
systemctl status docker
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「EC2 安裝 Docker」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「EC2 安裝 Docker」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「EC2 安裝 Docker」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「EC2 安裝 Docker」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「在遠端主機安裝 Docker 並確認權限與服務狀態」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 20 的重點是：用「EC2 安裝 Docker」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 21 - DAY 21 - 在 instance 上跑 Docker Image

來源：https://ithelp.ithome.com.tw/articles/10335418

### 這篇文章主要在講什麼

這篇聚焦在「EC2 跑 image」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：docker login 除了可以使用之前在 DAY 18 - 將我的 Image 推到 Docker Hub 儲存庫 ，教的 docker login ，再根據指示輸入 username 跟 password 外，也可以用下面一句話搞定。整理成工作筆記時，可以把它看成主線專案的「在 instance pull/run image，用 security group 與 curl 驗證對外服務」。

原文可辨識的重點標題包含：在 EC2 instance 登入 Docker Hub、docker login、docker login、docker login -u < 我的 docker username > -p < 我的 docker password >、Login Succeeded。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。EC2 跑 image 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：在 instance pull/run image，用 security group 與 curl 驗證對外服務。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- docker login 除了可以使用之前在 DAY 18 - 將我的 Image 推到 Docker Hub 儲存庫 ，教的 docker login ，再根據指示輸入 username 跟 password 外，也可以用下面一句話搞定。
- docker login -u < 我的 docker username > -p < 我的 docker password > 順利登入成功後，一樣可以看到 Login Succeeded。
- docker pull krystallll/docker_test:1.0 就跟我們 git pull 大同小異，只要打對 遠端儲存庫/image 名稱:tag 就可以順利 pull 下 image 啦。
- 延伸整理：這一天的核心不是背 `docker run -d -p 80:8080 yourname/deploy-demo:prod`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「EC2 跑 image」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「EC2 跑 image」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker run -d -p 80:8080 yourname/deploy-demo:prod
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「EC2 跑 image」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「EC2 跑 image」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「EC2 跑 image」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「EC2 跑 image」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「在 instance pull/run image，用 security group 與 curl 驗證對外服務」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 21 的重點是：用「EC2 跑 image」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 22 - DAY 22 - 在 EC2 instance 加上 docker compose 跑起來

來源：https://ithelp.ithome.com.tw/articles/10336096

### 這篇文章主要在講什麼

這篇聚焦在「EC2 跑 Compose」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：昨天我們實際讓 EC2 instance 使用 pull 下來的 image 啟動 container，因為遇到熟悉的錯誤，所以今天要來在 EC2 instance 加上 docker-compose.yml 跑起來。整理成工作筆記時，可以把它看成主線專案的「把 compose 檔與 env 放上 EC2，用一條命令啟動整組服務」。

原文可辨識的重點標題包含：建立 docker-compose.yml 檔案，並寫入資料、vim、1. 在對的路徑下創建、編輯 docker-compose.yml、vim docker-compose.yml、2. 寫入對的資料到 docker-compose.yml。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。EC2 跑 Compose 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：把 compose 檔與 env 放上 EC2，用一條命令啟動整組服務。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 昨天我們實際讓 EC2 instance 使用 pull 下來的 image 啟動 container，因為遇到熟悉的錯誤，所以今天要來在 EC2 instance 加上 docker-compose.yml 跑起來。
- 在終端機要建立、編輯檔案可以選擇自己熟悉的文本編輯器，我這邊是使用 vim 編輯器，若是對於一些 vim 編輯器的語法有興趣深入鑽研，可以另外再去找文章學習，這邊只會簡單介紹基礎 vim 語法。
- 一樣先進入到 EC2 instance 的終端機，使用 vim 指令建立加進去檔案編輯 vim docker-compose.yml 這串按下 enter 會出現如下的編輯畫面 我們可以按鍵盤 i 鍵來切換成輸入模式，並將本來專案的 docker-compose.yml 檔內容給貼上 ❗特別注意❗ 本來在本機我們是根據本機的 dockerfile 來建置 app 服務的容器(如下) app: build: context: . 但這邊我們要改使用我們在 Docker Hub 拉下來的 image ，來建立 app 容器，所以需要改成(如下) app: image: krystallll/docker_test:1.0 如此一來現在 docker-compose.yml 內容會長成如下： 剛剛使用鍵盤 i 進入輸入模式，當需要退出輸入模式時，可以使用 esc 鍵，這時試試如何敲打鍵盤都不會有字被打出來，這時我們需要儲存檔案並退出檔案，可以使用 :wq 鍵，是 儲存並退出 的意思。
- 延伸整理：這一天的核心不是背 `docker compose up -d / docker compose logs -f`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「EC2 跑 Compose」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「EC2 跑 Compose」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker compose up -d
docker compose logs -f
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「EC2 跑 Compose」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「EC2 跑 Compose」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「EC2 跑 Compose」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「EC2 跑 Compose」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「把 compose 檔與 env 放上 EC2，用一條命令啟動整組服務」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 22 的重點是：用「EC2 跑 Compose」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 23 - DAY 23 - 了解 Traefik 反向代理伺服器

來源：https://ithelp.ithome.com.tw/articles/10336768

### 這篇文章主要在講什麼

這篇聚焦在「Traefik」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：昨天已經部署完了就達成目的啦，怎麼我今天還出現。整理成工作筆記時，可以把它看成主線專案的「理解 reverse proxy、自動路由、entrypoints、providers、middleware」。

原文可辨識的重點標題包含：http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000、http://52.199.213.167:3000、不安全、將網址從 HTTP 變成 HTTPS 的步驟、1. 取得SSL/TLS憑證：。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Traefik 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：理解 reverse proxy、自動路由、entrypoints、providers、middleware。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 昨天已經部署完了就達成目的啦，怎麼我今天還出現。
- (開玩笑的)，其實是因為目前大概只能說是完成部署的 7 成，不知道大家有沒有發現，我們一直以來輸入的網址，無論是 DNS http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000 或是 IP Address http://52.199.213.167:3000 兩者的網址都是 HTTP ，但其實在瀏覽器也會顯示 HTTP 是個 不安全 的通訊協議 正常應該要是 HTTPS 的安全網址才是正確的，那我們應該要如何將網址從 HTTP 變成 HTTPS 呢。
- 在一個受信任的憑證授權單位（CA）（例如 Let's Encrypt、Comodo、DigiCert 等）來取得 SSL/TLS 憑證。
- 延伸整理：這一天的核心不是背 `docker compose logs traefik`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Traefik」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Traefik」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker compose logs traefik
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Traefik」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Traefik」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Traefik」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Traefik」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「理解 reverse proxy、自動路由、entrypoints、providers、middleware」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 23 的重點是：用「Traefik」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 24 - DAY 24 - 在我部署的專案使用 Traefik 取得 HTTPS 協定(一)

來源：https://ithelp.ithome.com.tw/articles/10337284

### 這篇文章主要在講什麼

這篇聚焦在「Traefik HTTPS 一」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：我參考 官網 用 Docker 啟動 Traefik ，跟者步驟試試 docker-compose down 確認一下有沒有容器在跑 docker ps 官網上面說在 docker-compose.yml 加上 reverse-proxy 這個服務 reverse-proxy: # The official v2 Traefik docker image image: traefik:v2.10 # Enables the web UI and tells Traefik to listen to docker command: --api.insecure=true --providers.docker ports: # The HTTP port - "80:80" # The Web UI (enabled by --api.insecure=true) - "8080:8080" volumes: # So that Traefik can listen to the Docker events - /var/run/docker.sock:/var/run/docker.sock 我根據上方官網的提示將 service reverse-proxy 改為 traefik ，然後重新啟動 docker-compose 一次，首先使用 vi 來編輯: vi docker-compose.yml 現在的 docker-compose.yml 長這樣： version: "3.9" services: app: image: krystallll/docker_test:1.0 environment: POSTGRES_USER: postgres POSTGRES_PASSWORD: password POSTGRES_HOST: db POSTGRES_PORT: 5432 restart: on-failure ports: - 3000:3000 db: image: postgres:14-alpine restart: on-failure environment: POSTGRES_PASSWORD: password traefik: image: traefik:v2.10 command: --api.insecure=true --providers.docker ports: - "80:80" - "8080:8080" volumes: - /var/run/docker.sock:/var/run/docker.sock docker-compose up --build 官網說可以去以下網址看 Traefik 的 API 原始資料 http://localhost:8080/api/rawdata 看到無法連上網站，我先盲猜是因為沒有開 8080 port，所以跟之前一樣我先去 AWS EC2 裡跟之前設定 3000 port 一樣去設定 8080 port。整理成工作筆記時，可以把它看成主線專案的「用 Traefik router/service/entrypoint 讓服務從 HTTP 入口被代理」。

原文可辨識的重點標題包含：在 docker-compose.yml 加上 traefik 服務、1. 先將原本的 docker-compose 停下、docker-compose down、docker ps、編輯 docker-compose.yml。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Traefik HTTPS 一 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：用 Traefik router/service/entrypoint 讓服務從 HTTP 入口被代理。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 我參考 官網 用 Docker 啟動 Traefik ，跟者步驟試試 docker-compose down 確認一下有沒有容器在跑 docker ps 官網上面說在 docker-compose.yml 加上 reverse-proxy 這個服務 reverse-proxy: # The official v2 Traefik docker image image: traefik:v2.10 # Enables the web UI and tells Traefik to listen to docker command: --api.insecure=true --providers.docker ports: # The HTTP port - "80:80" # The Web UI (enabled by --api.insecure=true) - "8080:8080" volumes: # So that Traefik can listen to the Docker events - /var/run/docker.sock:/var/run/docker.sock 我根據上方官網的提示將 service reverse-proxy 改為 traefik ，然後重新啟動 docker-compose 一次，首先使用 vi 來編輯: vi docker-compose.yml 現在的 docker-compose.yml 長這樣： version: "3.9" services: app: image: krystallll/docker_test:1.0 environment: POSTGRES_USER: postgres POSTGRES_PASSWORD: password POSTGRES_HOST: db POSTGRES_PORT: 5432 restart: on-failure ports: - 3000:3000 db: image: postgres:14-alpine restart: on-failure environment: POSTGRES_PASSWORD: password traefik: image: traefik:v2.10 command: --api.insecure=true --providers.docker ports: - "80:80" - "8080:8080" volumes: - /var/run/docker.sock:/var/run/docker.sock docker-compose up --build 官網說可以去以下網址看 Traefik 的 API 原始資料 http://localhost:8080/api/rawdata 看到無法連上網站，我先盲猜是因為沒有開 8080 port，所以跟之前一樣我先去 AWS EC2 裡跟之前設定 3000 port 一樣去設定 8080 port。
- 一樣先去編輯 Security Groups 加上 8080 port 重新看 http://localhost:8080/api/rawdata 一樣是無法連上網站，突然靈機一動上面的網址是 http://localhost 但現在網址是 http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com 所以我換網址為 http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:8080/api/rawdata 可以看到以下 終於成功看到 API 原始資料，那明天我們繼續來完成使用 Traefik 取得 HTTPS 協定。
- 延伸整理：這一天的核心不是背 `docker compose config`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Traefik HTTPS 一」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Traefik HTTPS 一」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker compose config
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Traefik HTTPS 一」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Traefik HTTPS 一」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Traefik HTTPS 一」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Traefik HTTPS 一」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「用 Traefik router/service/entrypoint 讓服務從 HTTP 入口被代理」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 24 的重點是：用「Traefik HTTPS 一」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 25 - DAY 25 - 在我部署的專案使用 Traefik 取得 HTTPS 協定(二)

來源：https://ithelp.ithome.com.tw/articles/10337821

### 這篇文章主要在講什麼

這篇聚焦在「Traefik HTTPS 二」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：昨天我們成功在 http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:8080/api/rawdata 成功看到 API 原始資料，那今天首先就先去 http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000 看看 因為昨天有 --build 的關係，所以會跑出這個需要我們建 db 是預料中的。整理成工作筆記時，可以把它看成主線專案的「加入 Let's Encrypt resolver 與 TLS 設定，讓憑證自動申請」。

原文可辨識的重點標題包含：http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:8080/api/rawdata、http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000、https://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000、將 app 服務跟 traefik 連結、docker-compose up。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Traefik HTTPS 二 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：加入 Let's Encrypt resolver 與 TLS 設定，讓憑證自動申請。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 昨天我們成功在 http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:8080/api/rawdata 成功看到 API 原始資料，那今天首先就先去 http://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000 看看 因為昨天有 --build 的關係，所以會跑出這個需要我們建 db 是預料中的。
- 到這邊為止目前看起來都沒有異常，但我們一開始使用 traefik 是因為我們需要將網址從 http 變成 https ，所以我們將網址改為 https://ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000 再試一次 去看看 log HTTP parse error, malformed request: #<Puma::HttpParserError: Invalid HTTP format, parsing fails. Are you trying to open an SSL connection to a non-SSL Puma?> 可以猜測這應該是一個無效的 http 請求，為甚麼會無效呢。
- 我重新看一下 官網 的圖片 可以看到交警伯伯 traefik ，當我們訪問網址時，會找到交警伯伯，但是我們好像還沒有將我們這個網址的服務 app 跟他做連結，所以我需要先將我的 app 服務與 traefik 做個連結 我找到一篇蒼時弦也的 Rails 部署實踐 - 使用 HTTPS 協定加密連線 與我要達成的目標相近，所以參考這篇文章並將其改為符合我專案的樣子，目前 docker-compose.yml 長成如下： version: "3.9" services: app: image: krystallll/docker_test:1.0 environment: POSTGRES_USER: postgres POSTGRES_PASSWORD: password POSTGRES_HOST: db POSTGRES_PORT: 5432 restart: on-failure # ports: # - 80:3000 labels: - "traefik.enable=true" - "traefik.http.services.app.loadbalancer.server.port=3000" - "traefik.http.routers.app.rule=Host(`ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com`)" - "traefik.http.routers.app.entrypoints=web" - "traefik.http.routers.app.tls=true" - "traefik.http.routers.app.tls.certresolver=letsencrypt" db: image: postgres:14-alpine restart: on-failure environment: POSTGRES_PASSWORD: password traefik: image: traefik:v2.10 command: - "--api.insecure=true" - "--providers.docker=true" - "--providers.docker.exposedbydefault=false" - "--entrypoints.web.address=:80" - "--entrypoints.websecure.address=:443" - "--certificatesresolvers.letsencrypt.acme.httpchallenge=true" - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web" - "--certificatesresolvers.letsencrypt.acme.email=krystal@5xcampus.com" - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json" ports: - "80:80" - "443:443" - "8080:8080" volumes: - "./letsencrypt:/letsencrypt" - /var/run/docker.sock:/var/run/docker.sock 這邊將原本 middlewares 有關的都先拿掉，一方面是因為我在下方的錯誤訊息裡撞牆了好久 ec2-user-traefik-1 | time="2023-10-10T17:54:19Z" level=error msg="middleware "https-redirect@docker" does not exist" entryPointName=web routerName=app-http@docker 另外也因為，希望是先用最簡化的方式教大家部署，一種先求有再求好的心態，所以我這邊就先移除 middlewares。
- 延伸整理：這一天的核心不是背 `docker compose logs traefik | findstr acme`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Traefik HTTPS 二」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Traefik HTTPS 二」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker compose logs traefik | findstr acme
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Traefik HTTPS 二」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Traefik HTTPS 二」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Traefik HTTPS 二」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Traefik HTTPS 二」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「加入 Let's Encrypt resolver 與 TLS 設定，讓憑證自動申請」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 25 的重點是：用「Traefik HTTPS 二」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 26 - DAY 26 - 要取得協定得先有正確的 Domain

來源：https://ithelp.ithome.com.tw/articles/10338280

### 這篇文章主要在講什麼

這篇聚焦在「Domain」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：昨天結尾的錯誤訊息説，無法發給 ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com 這個 domain 正確的 ACME 證書。整理成工作筆記時，可以把它看成主線專案的「讓 DNS A record 指向 EC2 public IP，憑證申請前先確認 domain 解析」。

原文可辨識的重點標題包含：ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com、mydocker.online、在 Godaddy 管理 DNS、1. 點選右上角頭像到我的產品、2. 點選這個 domain 的 DNS 按鈕。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Domain 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：讓 DNS A record 指向 EC2 public IP，憑證申請前先確認 domain 解析。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 昨天結尾的錯誤訊息説，無法發給 ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com 這個 domain 正確的 ACME 證書。
- 所以推測可能我們使用的 AWS DNS 並沒有符合 Let's Encrypt ，所以我們會需要一個正常的 Domain ，那我這邊是直接在 Godaddy 選購，我買了一個 mydocker.online 的 Domain 購買完成後，還需要在 Godaddy 設定 AWS Public IP address 以下步驟我是參考從谷歌大神那邊問來的 文章 Ａ @ 等待的同時我們可以先去本來的專案，將 development.rb 之前設定的 config.hosts << "ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com:3000" unless Rails.root.join("tmp/hostname.txt").exist? 改為 config.hosts << "mydocker.online" unless Rails.root.join("tmp/hostname.txt").exist? 因為我們現在有換新的 domain 所以這個才需要修改，然後重新 build image 推上 Docker Hub (記得先 login) docker build -t krystallll/docker_test:1.0 . --push 在 EC2 inatance 裡，先 login 到 Docker Hub，再將最新的 image 拉下來 docker pull krystallll/docker_test:1.0 將本來的 - "traefik.http.routers.app.rule=Host(`ec2-52-199-213-167.ap-northeast-1.compute.amazonaws.com`)" 改為 - "traefik.http.routers.app.rule=Host(`mydocker.online`)"` 並且將 app 的 ports 改為 ports: - 80:3000 在 docker 的 80 port 對到 app 的 3000 port，並先把 traefik 服務都先註解掉，我先確認這個 domain 確定可以連接，這時使用 docker-compose up 📍我們在設定 Godaddy 選擇了 600 秒，代表 600 秒後才會生效，所以若沒畫面也可能是還沒有成功生效，他最晚一天之內一定會好，如果沒好那可能就是你沒設定好🥺 既然 Domain 都用好了，明天又要重回 traefik debug 地獄🤮🤮🤮。
- 延伸整理：這一天的核心不是背 `nslookup example.com`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Domain」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Domain」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
nslookup example.com
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Domain」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Domain」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Domain」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Domain」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「讓 DNS A record 指向 EC2 public IP，憑證申請前先確認 domain 解析」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 26 的重點是：用「Domain」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 27 - DAY 27 - 在我部署的專案使用 Traefik 取得 HTTPS 協定(三)

來源：https://ithelp.ithome.com.tw/articles/10338739

### 這篇文章主要在講什麼

這篇聚焦在「Traefik HTTPS 三」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：昨天將 domain 設定好，確定目前的 mydocker.online 是已經跟我們的服務連上的， docker-compose.yml 目前長成如下，那我們今天要一步步來除錯。整理成工作筆記時，可以把它看成主線專案的「整合 domain、Traefik、Compose 與 app label，完成 HTTPS 對外服務」。

原文可辨識的重點標題包含：mydocker.online、docker-compose.yml、ports: - 80:3000、docker-compose up、https://mydocker.online。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Traefik HTTPS 三 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：整合 domain、Traefik、Compose 與 app label，完成 HTTPS 對外服務。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 昨天將 domain 設定好，確定目前的 mydocker.online 是已經跟我們的服務連上的， docker-compose.yml 目前長成如下，那我們今天要一步步來除錯。
- version: "3.9" services: app: image: krystallll/docker_test:1.0 environment: POSTGRES_USER: postgres POSTGRES_PASSWORD: password POSTGRES_HOST: db POSTGRES_PORT: 5432 restart: on-failure ports: - 80:3000 labels: - "traefik.enable=true" - "traefik.http.services.app.loadbalancer.server.port=3000" - "traefik.http.routers.app.rule=Host(`mydocker.online`)"` - "traefik.http.routers.app.entrypoints=web" - "traefik.http.routers.app.tls=true" - "traefik.http.routers.app.tls.certresolver=letsencrypt" db: image: postgres:14-alpine restart: on-failure environment: POSTGRES_PASSWORD: password # traefik: # image: traefik:v2.10 # command: # - "--api.insecure=true" # - "--providers.docker=true" # - "--providers.docker.exposedbydefault=false" # - "--entrypoints.web.address=:80" # - "--entrypoints.websecure.address=:443" # - "--certificatesresolvers.letsencrypt.acme.httpchallenge=true" # - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web" # - "--certificatesresolvers.letsencrypt.acme.email=krystal@5xcampus.com" # - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json" # ports: # - "80:80" # - "443:443" # - "8080:8080" # volumes: # - "./letsencrypt:/letsencrypt" # - /var/run/docker.sock:/var/run/docker.sock 當我將上述註解都打開，並且移除 app 服務的 ports: - 80:3000 (移除是因為 traefik 服務有 80 port ，所以可以由他指揮去 app 服務) 後，重新 docker-compose up ，再去網址 https://mydocker.online 時，不外乎一樣是 404 page not found。
- 統整一下我搜集到的可能連接不上的所有原因，來一一檢查修改 因為我的 docker-compose.yml 沒有特別設定在哪個網路下執行，所以我們就先來設定一下，確保我的 docker-compose 的所有服務都在同個網路下執行，可以互相通信，如果大家對於 Network 不太熟悉，可以參考 DAY 6 Docker Network(網路)容器與容器間的橋樑 首先我要先建立一個網路，我就取名為 development 好了，那就在我的 AWS EC2 instance 終端機裡面，我可以先確認現在的 Network 有哪些 docker network ls 再來建立名為 development 的網路 docker network create development 看到這串 SHA 值就代表建立成功。
- 延伸整理：這一天的核心不是背 `curl -I https://example.com`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Traefik HTTPS 三」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Traefik HTTPS 三」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
curl -I https://example.com
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Traefik HTTPS 三」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Traefik HTTPS 三」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Traefik HTTPS 三」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Traefik HTTPS 三」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「整合 domain、Traefik、Compose 與 app label，完成 HTTPS 對外服務」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 27 的重點是：用「Traefik HTTPS 三」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 28 - DAY 28 - 使用 Volumes 讓我的資料庫保存之前的資料

來源：https://ithelp.ithome.com.tw/articles/10339164

### 這篇文章主要在講什麼

這篇聚焦在「Volumes」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：已經部署完成了 我們現在就混水摸魚等鐵人賽 30 天到(開玩笑的) ，今天還有什麼好介紹的呢。整理成工作筆記時，可以把它看成主線專案的「用 volume 保存資料庫與憑證資料，不讓 container 重建就清空狀態」。

原文可辨識的重點標題包含：docker-compose down、docker-compose up、db:
    image: postgres:14-alpine
    restart: on-failure
    environment:
      POSTGRES_PASSWORD: password、docker-compose db save data、Volumes 是做什麼的？。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Volumes 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：用 volume 保存資料庫與憑證資料，不讓 container 重建就清空狀態。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 已經部署完成了 我們現在就混水摸魚等鐵人賽 30 天到(開玩笑的) ，今天還有什麼好介紹的呢。
- 其實是我昨天發現了一個部署的大 bug ，就是當我每次 docker-compose down 後，再次 docker-compose up 都會需要重新 create db 等於我每次的資料都沒留下來 基本上比本機還不如 😅 那今天就立馬來解決這個 bug。
- 我首先先打開我久違的專案 (終於暫時脫離 AWS EC2) ，一想到 docker 裡跟 database 有關的設定，我就先去 docker-compose.yml 的 postgresql 服務裡看起： db: image: postgres:14-alpine restart: on-failure environment: POSTGRES_PASSWORD: password 於是我下關鍵字 docker-compose db save data 問最愛的谷歌大神，找到第一個搜尋結果 stackoverflow ，就看到疑似答案的東西，就是需要加上 volumes。
- 延伸整理：這一天的核心不是背 `docker volume ls / docker volume inspect app_db_data`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Volumes」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Volumes」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker volume ls
docker volume inspect app_db_data
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Volumes」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Volumes」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Volumes」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Volumes」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「用 volume 保存資料庫與憑證資料，不讓 container 重建就清空狀態」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 28 的重點是：用「Volumes」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 29 - DAY 29 - 將 Dockerfile 改成多階段建置

來源：https://ithelp.ithome.com.tw/articles/10339632

### 這篇文章主要在講什麼

這篇聚焦在「Multi-stage build」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：多階段建置可以在一開始建置階段使用標準版本的鏡像，讓建置階段時，就可以下載好需要的工具和依賴套件，然後在第二階段時，就可以換到輕量級的 Alpine 鏡像。整理成工作筆記時，可以把它看成主線專案的「用 build stage 與 runtime stage 減少 image 大小與攻擊面」。

原文可辨識的重點標題包含：多階段建置的優點、減少鏡像大小：、提高安全性：、最佳化建置速度：、更清晰的建置流程：。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Multi-stage build 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：用 build stage 與 runtime stage 減少 image 大小與攻擊面。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 多階段建置可以在一開始建置階段使用標準版本的鏡像，讓建置階段時，就可以下載好需要的工具和依賴套件，然後在第二階段時，就可以換到輕量級的 Alpine 鏡像。
- 如此一來可以將應用程式，以及只包含執行時所需的依賴套件，可以不包括多餘的檔案，可以減小鏡像的大小。
- 當我們可以減少容器的鏡像大小，就代表著減少了潛在的風險，所以可以提高安全性。
- 延伸整理：這一天的核心不是背 `docker build -t deploy-demo:multi . / docker image ls deploy-demo`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Multi-stage build」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Multi-stage build」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker build -t deploy-demo:multi .
docker image ls deploy-demo
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Multi-stage build」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Multi-stage build」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Multi-stage build」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Multi-stage build」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「用 build stage 與 runtime stage 減少 image 大小與攻擊面」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 29 的重點是：用「Multi-stage build」把主線專案往可部署、可驗證、可交接的方向推進一步。

## Day 30 - DAY 30 - 將 docker-compose.yml 裡的 container 取名

來源：https://ithelp.ithome.com.tw/articles/10339949

### 這篇文章主要在講什麼

這篇聚焦在「Container 命名」。根據原文脈絡，它把前一天的基礎往部署流程推進一步：昨天整理了 Dockerfile ，今天換 docker-compose.yml 吧。整理成工作筆記時，可以把它看成主線專案的「用 container_name、service name、hostname 規劃可讀的部署命名」。

原文可辨識的重點標題包含：自訂 container 名稱、docker-compose up、docker_test-db-1、docker_test-app-1、專案名稱＋服務名稱。

### 為什麼需要這個概念

因為部署不是單一指令，而是一串可追蹤的狀態變化。Container 命名 讓你知道現在是在處理本機環境、image、container、network、遠端主機、反向代理，還是資料保存。沒有這層判斷，錯誤發生時很容易只重跑指令，卻不知道該查 Docker、雲端、防火牆、DNS 還是 app log。

### 學完這篇你應該會做到什麼

你應該能把這一天的主題落成一個可驗證交付物：用 container_name、service name、hostname 規劃可讀的部署命名。至少要能說出輸入、會改到哪個 Docker 狀態、成功時要看哪個輸出，以及失敗時第一個排查位置。

### 核心重點

- 昨天整理了 Dockerfile ，今天換 docker-compose.yml 吧。
- 不知道大家有沒有注意到．每次 docker-compose up 時，看到的畫面 這些是他自動給容器的名稱 docker_test-db-1 或 docker_test-app-1 ，因為我們沒有幫容器取名，所以他就根據 專案名稱＋服務名稱 來命名。
- 但是若你不喜歡這個名稱想自定義名稱也是可以的，只要在服務後加上 container_name: <你想取的名字> 在 docker-compose up 時就可以看到 成功將名稱更換了，如此一來我們更可以透過 log 知道應該在哪個容器除錯。
- 延伸整理：這一天的核心不是背 `docker compose ps`，而是理解它在部署流程中改變了什麼。

### 真實工作流程例子

- 工作任務：團隊要把主線 Web 專案推進到「Container 命名」這一階段，並產出可以交給同事或部署環境驗證的結果。
- 你先判斷：先確認這次是在改本機 Docker 操作、`Dockerfile`、`docker-compose.yml`、EC2 主機、Traefik 設定、DNS，還是資料保存；不要一開始就複製指令，因為錯層修改會讓錯誤訊息更混亂。
- 會動到：`Dockerfile`、`docker-compose.yml`、`.env.example`、EC2 shell、Docker Desktop / Docker CLI、Traefik labels、DNS console 中與「Container 命名」相關的部分。
- 資料怎麼流：使用者或部署需求進來後，先轉成設定或指令；Docker CLI 讀取本機檔案與參數，建立 image / container / network / volume；部署階段再由 EC2、Traefik、Domain 把外部 request 導到 app container。
- 流程路線圖：需求 -> 判斷責任層 -> 修改設定或執行指令 -> Docker / EC2 狀態改變 -> 透過 CLI、log、HTTP、DNS 或 volume 驗證結果。
- 工作中會寫 / 檢查的片段：

```bash
# 範例用途：用這一天的主題建立或檢查部署流程中的一個狀態。
# 主要輸入：專案檔案、image 名稱、container 名稱、port、domain 或遠端主機資訊。
# 預期結果：命令成功後，可以用 ps/log/curl/DNS/volume inspect 看到可驗證狀態。
docker compose ps
```

- 交付前驗證：確認命令 exit code 成功；用 `docker ps`、`docker logs`、`docker compose ps`、`curl` 或 DNS 查詢驗證；再測一次失敗情境，例如 port 沒開、image tag 不存在、domain 尚未解析、volume 沒掛載。
- 常見卡點：junior 常把錯誤都歸因於 Docker，但這一天要先查的是「Container 命名」所在層；第一步看命令輸出與 log，第二步才檢查設定檔或雲端介面。

### 主線專案銜接

今天接到主線專案的「Container 命名」段落。

具體流程：
1. 先讀本日來源文章，確認它對應到主線部署流程的哪個責任層。
2. 在主線專案中新增、修改或檢查與「Container 命名」相關的檔案或平台設定。
3. 執行本日命令，保留輸出結果，並用至少一個獨立驗證方式確認狀態。

具體檢查：
- 檔案或平台設定是否放在正確位置。
- 命令是否在正確工作目錄或正確遠端主機執行。
- 驗證結果是否能證明這一天的交付物真的完成。

### 當天做完後檢查

- 可以用自己的話說明今天處理的是哪個部署責任層。
- 可以重跑本日主要命令，並知道成功輸出大概長什麼樣子。
- 可以指出一個常見失敗原因與第一個排查命令。

### 範例範圍地圖

| 部分 | 位置 / 檔案 / 指令 / 介面 | 負責的事情 |
| --- | --- | --- |
| 資料或設定 | `.env、docker-compose.yml、Dockerfile、DNS console` | 提供 image、port、環境變數、domain 或 volume 設定 |
| 流程或邏輯 | `Docker CLI、docker compose、EC2 shell、Traefik provider` | 依設定建立、啟動、代理或保存服務狀態 |
| 使用端或呈現 | `curl、browser、docker logs、Docker Desktop` | 確認 app 是否能被呼叫，錯誤是否能被追蹤 |

### 完整實作流程、範例與注意事項

完整流程：

1. 確認今天的主題是「Container 命名」，先不要跨到還沒準備好的部署層。
2. 檢查目前工作目錄、Docker daemon、遠端主機或 DNS 狀態是否符合前置條件。
3. 修改必要檔案或執行本日命令。
4. 用另一個角度驗證，例如跑完 build 後看 image、跑完 compose 後看 service、設定 domain 後查 DNS。
5. 把成功輸出與失敗排查步驟寫進團隊交接文件。

注意事項：
- 不要把 demo 指令直接貼到正式環境；先替 image、container、volume、domain 命名。
- 如果命令會建立雲端資源或公開服務，要先確認成本、安全群組與清理方式。
- 若涉及 secret，筆記只保留 placeholder，不提交真實密碼或 token。

### 如果結果和預期不同

- 先看命令最後 20 行輸出，不要只看第一個錯誤字。
- 用 `docker ps -a` 確認 container 是沒建立、已退出，還是正在跑但無法連。
- 若是遠端部署，分別檢查 EC2 security group、主機防火牆、container port mapping、Traefik router、DNS。

### 負面例子 / 錯誤用法

錯誤做法：看到網路上某段指令就直接在正式主機執行，沒有確認 image tag、volume、port、secret 與清理方式。

問題：
- 可能覆蓋正在跑的服務或把資料寫到暫時 container。
- 發生錯誤時不知道該從 Docker、EC2、Traefik 還是 DNS 查起。
- 使用 `latest` 或未命名資源會讓回滾與交接變困難。

修正方向：先把命令變成可讀的部署步驟，命名 image / container / volume，並寫下驗證命令。

### 小練習

把今天的主題套到一個最小 Web API：先完成「用 container_name、service name、hostname 規劃可讀的部署命名」，再故意製造一個錯誤，例如 port 寫錯、image tag 打錯、domain 尚未指向，練習用 log 與 CLI 找出問題。

### Junior 常見誤解

常見誤解是把「指令成功執行」當成「部署完成」。真正的完成是有一個外部可驗證結果，例如 HTTP 回應、container 狀態、log、DNS 解析或 volume 資料仍存在。

### 一句話總結

Day 30 的重點是：用「Container 命名」把主線專案往可部署、可驗證、可交接的方向推進一步。
