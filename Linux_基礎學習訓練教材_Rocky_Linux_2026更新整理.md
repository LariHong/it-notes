# Linux 基礎學習訓練教材：Rocky Linux 2026 更新整理

## 來源

- 鳥哥 Linux 基礎學習訓練教材 CentOS 7.x 目錄：https://linux.vbird.org/linux_basic_train/centos7/
- 鳥哥 Linux 基礎學習訓練教材 CentOS 8.x 目錄：https://linux.vbird.org/linux_basic_train/centos8/
- Red Hat CentOS Linux EOL 說明：https://www.redhat.com/en/blog/centos-linux-end-life-centos-stream-and-new-red-hat-enterprise-linux-landscape
- Red Hat Customer Portal：CentOS Linux end of life：https://access.redhat.com/articles/7120228
- Rocky Linux Release and Version Guide：https://wiki.rockylinux.org/rocky/version/

## 來源提醒與 2026 版本選擇

鳥哥這份教材原本是 CentOS 7.x 授課目錄，頁面標示最近更新時間是 2017/06/29；CentOS 8.x 訓練教材則標示到 2020/06/24。以 2026 年來看，CentOS Linux 7 已在 2024/06/30 結束生命週期，Red Hat 也說明 CentOS Linux 不再收到安全修補、功能更新或 bug fix。Rocky Linux 官方版本頁面列出 Rocky Linux 9 支援到 2032/05/31，Rocky Linux 10 支援到 2035/05/31。

因此這份筆記採用這個策略：

- 觀念主軸沿用鳥哥 15 堂課的學習順序。
- 實作環境建議使用 Rocky Linux 9 或 Rocky Linux 10；若公司環境是 RHEL 9/10、AlmaLinux 9/10，大多數基礎命令也可套用。
- CentOS 7 的 `yum`、舊網卡命名、舊服務設定方式只當歷史脈絡補充；練習時優先使用 `dnf`、`systemctl`、`nmcli`、`firewall-cmd` 等現代 RHEL 系工具。
- 你從未用過 Linux 的話，不要先追求背指令；請先建立「檔案、使用者、權限、程序、服務、套件、網路」這幾個模型。

## 這份筆記怎麼讀

建議先準備一台可丟掉重建的虛擬機器，不要直接拿工作機或正式伺服器練習。Linux 學習最有效的方式是「看概念、打一遍、故意犯錯、查狀態、修回來」。

最低練習環境：

- VirtualBox、VMware Workstation Player、Hyper-V、WSL2 或雲端 VM 皆可。
- 建議安裝 Rocky Linux 9 minimal 或 Rocky Linux 10 minimal。
- 建議 VM 規格：2 CPU、2 GB RAM、30 GB disk。
- 建議建立一般使用者，例如 `student`，平常不要直接用 `root` 登入。

安全提醒：

- 任何 `rm -rf`、`mkfs`、`fdisk`、`parted`、`dd` 都可能破壞資料。
- 練習磁碟與檔案系統前，先拍 VM snapshot。
- 看到網路文章叫你關閉 SELinux 或防火牆，不要照抄；先學會查原因。

每堂課請照這個節奏讀：

1. 先看「學完這堂你應該會做到什麼」，知道這堂課的成果。
2. 再看「指令地圖」，分清楚哪些命令只是查詢、哪些命令會修改系統。
3. 進入「實作任務情境」，把指令放進接近工作的問題裡理解。
4. 照著 step-by-step 操作，每一步都先知道目的，再執行命令。
5. 做完後一定要驗證結果；Linux 不是打完命令就算完成，能查證才算完成。
6. 如果結果不同，先看排查提示，不要急著重裝或亂改權限。

## 整體地圖

| 課程 | 主題 | 你要學會的能力 | 2026 實務對應 |
| --- | --- | --- | --- |
| 第 1 堂 | 初次使用 Linux 與指令列 | 知道 Linux 是作業系統，會登入、查基本資訊 | VM、雲端主機、SSH |
| 第 2 堂 | 指令下達與基礎檔案管理 | 會移動目錄、建立/複製/刪除檔案 | 每天查 log、部署檔案 |
| 第 3 堂 | 檔案管理與 vim | 會搜尋檔案、看內容、修改設定檔 | 改設定、看錯誤 |
| 第 4 堂 | 權限與帳號 | 看懂 owner/group/mode | 最小權限管理 |
| 第 5 堂 | 權限應用與程序 | 會看 process、停止異常程序 | 排查 CPU/RAM 問題 |
| 第 6 堂 | 檔案系統 | 會看磁碟、掛載、修復概念 | 空間不足處理 |
| 第 7 堂 | bash 與救援 | 會用 shell 變數、救援模式 | 修壞掉的系統 |
| 第 8 堂 | 串接指令與重導向 | 會 pipe、redirect、tee | 自動化前置能力 |
| 第 9 堂 | regex 與 shell script | 會 grep 入門、寫 script | 批次維運 |
| 第 10 堂 | 使用者、sudo、ACL | 建立帳號、控管 sudo 與細部權限 | 團隊共管主機 |
| 第 11 堂 | 基礎設定、備份、排程 | 時區、壓縮、cron/systemd timer | 例行備份 |
| 第 12 堂 | 套件與 log | 使用 dnf/rpm、查 journal | 安裝與排錯 |
| 第 13 堂 | 服務與開機流程 | 管理 systemd service | 讓服務穩定啟動 |
| 第 14 堂 | RAID、LVM、Quota | 理解儲存擴充與限制 | 伺服器磁碟規劃 |
| 第 15 堂 | 系統準備 | 安裝後基線設定 | 建立可維護的主機 |

---

## 第 1 堂：初次使用 Linux 與指令列模式初探

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit01.php

### 這堂課主要在講什麼

Linux 是負責管理硬體與提供程式執行環境的作業系統。剛開始不要把它想成「很多黑底白字指令」，而是把它想成一間機房的總管：CPU、記憶體、磁碟、網路都歸它調度，使用者透過 shell 對它下指令。

### 學完這堂你應該會做到什麼

- 可以說出 kernel、shell、distribution、terminal 的差異。
- 可以登入一台 Rocky Linux 主機，確認自己是誰、在哪台機器、目前在哪個目錄。
- 可以查出 Rocky Linux 版本與 kernel 版本。
- 可以判斷自己是不是不小心用 `root` 在操作。

### 為什麼需要這個概念

如果不知道「作業系統、核心、shell、發行版」的差異，後面會很容易混亂。例如你可能以為 `bash` 就是 Linux，或以為 Rocky Linux、Ubuntu、Debian 是不同核心。實際上 Linux kernel 是核心，Rocky Linux 是把 kernel、GNU 工具、套件管理、服務管理、預設設定包起來的 distribution。

### 核心重點

- Linux kernel 管硬體與核心資源。
- shell 是你和系統溝通的命令介面，常見是 `bash`。
- distribution 是完整作業系統包裝，例如 Rocky Linux、RHEL、Ubuntu。
- 初學者要先會登入、查身份、查目錄、離開系統。

### 定義名稱對照表

| 角色 | 本課名稱 | 功能 / 責任 |
| --- | --- | --- |
| Kernel | Linux kernel | 管理 CPU、記憶體、磁碟、網路與系統呼叫 |
| Shell | bash | 接收使用者命令並呼叫系統工具 |
| Distribution | Rocky Linux / RHEL 系 | 包裝 kernel、工具、套件庫與預設設定 |
| User | 一般使用者 / root | 代表操作系統的人或服務身份 |

### 直觀例子

把 Linux 想成餐廳廚房：kernel 是廚房總管，決定爐子、冰箱、刀具怎麼分配；shell 是櫃台，你對櫃台說「我要查菜單」，櫃台再叫後場處理；distribution 是整間餐廳的營運包裝，包含菜單、流程、制服與管理規則。

### 指令地圖

| 指令 | 你在查什麼 | 會不會修改系統 | 什麼時候會用到 |
| --- | --- | --- | --- |
| `whoami` | 目前登入身份 | 不會 | 確認自己是不是 `root` 或一般使用者 |
| `hostname` | 主機名稱 | 不會 | 確認目前連到哪台 VM / server |
| `pwd` | 目前所在目錄 | 不會 | 下任何檔案操作前先確認位置 |
| `cat /etc/os-release` | 發行版資訊 | 不會 | 確認 Rocky / RHEL / Ubuntu 等版本 |
| `uname -r` | kernel 版本 | 不會 | 查驅動、系統相容性、問題排查 |

### 實作任務情境

你剛拿到一台新的 Rocky Linux VM，前輩要你先確認「這台機器是什麼系統、你用什麼身份登入、目前在哪裡」。這是所有系統操作的第一個 checkpoint。

### 操作前檢查

- 你已經能開啟 VM 終端機，或能用 SSH 登入。
- 先不要切換成 `root`，用一般使用者練習。
- 這一組命令都是查詢型命令，不會修改系統。

### Step-by-step 實作

```bash
# 範例用途：確認自己登入的是哪台機器、哪個使用者、目前在哪個目錄。
# 輸入說明：直接在終端機輸入；不需要 root 權限。
# 輸出結果：會看到使用者、主機名稱、目前路徑與系統版本。
whoami
hostname
pwd
cat /etc/os-release
uname -r
```

1. 執行 `whoami`，確認目前使用者。預期會看到像 `student` 這樣的一般帳號；如果看到 `root`，代表你正在用最高權限操作。
2. 執行 `hostname`，確認目前主機名稱。預期會看到 VM 或雲端主機名稱，例如 `rocky-lab`。
3. 執行 `pwd`，確認目前路徑。剛登入時通常會在 `/home/student` 這類家目錄。
4. 執行 `cat /etc/os-release`，確認發行版。預期會看到 `Rocky Linux`、`VERSION_ID` 等欄位。
5. 執行 `uname -r`，確認 kernel 版本。這不是 Rocky Linux 版本，而是 Linux kernel 的版本。

### 如果結果和預期不同

- `whoami` 顯示 `root`：先輸入 `exit` 回到一般使用者，或重新用一般帳號登入。
- `cat /etc/os-release` 顯示 Ubuntu / Debian：代表你不是在 Rocky Linux 環境，後面 `dnf`、`firewall-cmd` 等行為可能不同。
- `pwd` 顯示 `/` 或 `/etc`：先不要建立或刪除檔案，回到家目錄再練習：`cd "$HOME"`。

### 做完後檢查

- 你能在筆記中寫下目前使用者、主機名稱、發行版名稱、發行版版本、kernel 版本。
- 你能解釋 `Rocky Linux 9/10` 和 `Linux kernel` 不是同一件事。

### 負面例子 / 錯誤用法

錯誤做法：一開始就用 `root` 當日常帳號，所有操作都在最高權限下進行。

問題：

- 打錯刪除命令時可能直接毀掉系統。
- 看不出哪些命令真的需要權限。
- 未來接觸正式主機時容易養成危險習慣。

修正方向：平常使用一般帳號，需要管理權限時才用 `sudo`。

### 小練習

1. 重新開一個終端機，再執行一次 `whoami`、`hostname`、`pwd`。
2. 用自己的話寫一句：「我現在用誰，在哪台主機，在哪個目錄操作。」
3. 試著分辨哪些輸出和「使用者」有關，哪些和「作業系統版本」有關。

### Junior 常見誤解

- `Linux` 不等於 `Ubuntu`，Ubuntu 只是 Linux 發行版之一。
- `terminal` 不等於 `shell`；terminal 是視窗或連線介面，shell 是裡面跑的命令解譯器。
- `root` 很方便，但不是練習時的預設身份。

### 一句話總結

Linux 入門的第一步不是背命令，而是知道你正在用哪個身份，對哪台系統，透過哪個 shell 下指令。

---

## 第 2 堂：指令下達行為與基礎檔案管理

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit02.php

### 這堂課主要在講什麼

Linux 的檔案系統像一棵樹，最上層是 `/`，所有檔案、硬碟、設定、裝置都掛在這棵樹底下。你每天最常做的事情就是移動位置、列出檔案、建立目錄、複製、搬移、刪除。

### 學完這堂你應該會做到什麼

- 可以分辨絕對路徑與相對路徑。
- 可以在家目錄建立練習資料夾與文字檔。
- 可以複製、改名、列出檔案，並用 `pwd` / `ls` 驗證結果。
- 可以知道 `rm` 是真的刪除，不是丟到垃圾桶。

### 為什麼需要這個概念

如果不熟檔案路徑，後面改設定、查 log、部署程式都會卡住。Linux 沒有 Windows 的 C 槽、D 槽概念，而是統一從 `/` 往下找。

### 核心重點

- 絕對路徑從 `/` 開始，例如 `/etc/ssh/sshd_config`。
- 相對路徑從目前位置開始，例如 `../logs/app.log`。
- `ls` 看檔案，`cd` 切目錄，`mkdir` 建目錄，`cp` 複製，`mv` 搬移或改名，`rm` 刪除。
- 刪除前先 `ls` 或 `pwd` 確認位置。

### 定義名稱對照表

| 角色 | 本課名稱 | 功能 / 責任 |
| --- | --- | --- |
| Root directory | `/` | Linux 檔案樹最上層 |
| Home directory | `/home/student` | 使用者自己的工作空間 |
| Command | `ls`, `cd`, `cp`, `mv`, `rm` | 操作檔案與目錄 |
| Path | absolute / relative path | 指向檔案位置 |

### 指令地圖

| 指令 | 用途 | 會不會修改系統 | Junior 要注意什麼 |
| --- | --- | --- | --- |
| `pwd` | 顯示目前目錄 | 不會 | 操作檔案前先看自己在哪裡 |
| `cd` | 切換目錄 | 不會直接修改檔案 | 相對路徑會受到目前目錄影響 |
| `mkdir -p` | 建立目錄 | 會 | `-p` 可建立多層目錄，已存在也不報錯 |
| `printf > file` | 建立或覆蓋檔案 | 會 | `>` 會覆蓋原本內容 |
| `cp` | 複製檔案 | 會 | 目標檔已存在時可能覆蓋 |
| `mv` | 搬移或改名 | 會 | 常用來改檔名 |
| `ls -l` | 列出詳細資訊 | 不會 | 用來驗證檔案是否存在與權限 |
| `rm` | 刪除檔案 | 會，而且通常不能復原 | 初學先搭配 `-i` 互動確認 |

### 實作任務情境

你要在自己的家目錄建立一個練習區，產生一份文字檔，複製成備份，再把備份改成有年份的檔名。這是部署、備份、整理 log 前最基本的檔案操作流程。

### 操作前檢查

- 確認你在一般使用者帳號，不要用 `root`。
- 先執行 `cd "$HOME"`，把練習範圍限制在自己的家目錄。
- 這個練習會建立與改名檔案，但不會碰系統設定。

### Step-by-step 實作

```bash
# 範例用途：建立一個練習資料夾，複製檔案，再改名。
# 輸入說明：$HOME 是目前使用者家目錄，不需要硬背完整路徑。
# 輸出結果：會建立 linux-practice 目錄與 hello.txt / hello-2026.txt。
cd "$HOME"
mkdir -p linux-practice
cd linux-practice
printf "hello linux\n" > hello.txt
cp hello.txt hello-backup.txt
mv hello-backup.txt hello-2026.txt
ls -l
```

1. 執行 `cd "$HOME"`，回到自己的家目錄。這一步是為了避免在 `/etc`、`/var` 或其他系統路徑亂建檔案。
2. 執行 `mkdir -p linux-practice`，建立練習資料夾。`-p` 讓命令在資料夾已存在時不報錯。
3. 執行 `cd linux-practice`，進入練習資料夾。
4. 執行 `printf "hello linux\n" > hello.txt`，建立一個文字檔。注意 `>` 會覆蓋同名檔案。
5. 執行 `cp hello.txt hello-backup.txt`，複製一份備份。
6. 執行 `mv hello-backup.txt hello-2026.txt`，把備份改名。
7. 執行 `ls -l`，確認目錄中有 `hello.txt` 和 `hello-2026.txt`。

### 如果結果和預期不同

- `cd linux-practice` 顯示 `No such file or directory`：代表前一步 `mkdir` 沒有成功，先回頭看命令是否打錯。
- `ls -l` 只看到 `hello.txt`：可能是 `cp` 或 `mv` 打錯檔名。
- `Permission denied`：你可能不在自己的家目錄，先執行 `pwd` 確認位置。

### 做完後檢查

```bash
pwd
ls -l
cat hello.txt
```

你應該能確認目前路徑在 `linux-practice`，並且 `hello.txt` 內容是 `hello linux`。

### 負面例子 / 錯誤用法

錯誤做法：

```bash
rm -rf *
```

問題：如果你剛好在錯的目錄，這會刪掉目前目錄下所有東西，而且通常沒有垃圾桶可以復原。

修正方向：

```bash
pwd
ls -la
rm -i hello-2026.txt
```

先確認位置，再用互動模式刪除重要檔案。

### 小練習

1. 再建立一個 `notes.txt`，內容寫入 `practice path`。
2. 把 `notes.txt` 複製成 `notes-backup.txt`。
3. 用 `ls -l` 驗證兩個檔案都存在。
4. 刪除前先執行 `pwd`，確認自己真的在 `linux-practice`。

### Junior 常見誤解

- `mv` 不只是搬移，也常被用來改名。
- Linux 檔名大小寫不同，`Readme.md` 和 `README.md` 是不同檔案。
- `.` 代表目前目錄，`..` 代表上一層。

### 一句話總結

檔案管理是 Linux 的基本步法，先站穩路徑，再動手操作。

---

## 第 3 堂：檔案管理與 vim 初探

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit03.php

### 這堂課主要在講什麼

這堂課把「找檔案、看檔案、編輯檔案」串起來。系統管理很大一部分是在修改設定檔，而設定檔大多是純文字，所以你要會用 `cat`、`less`、`head`、`tail`、`grep` 與至少一個文字編輯器。

### 學完這堂你應該會做到什麼

- 可以用 `cat`、`less`、`head`、`tail` 看文字檔內容。
- 可以用 `grep` 從檔案中找出特定關鍵字。
- 可以用 vim 建立或修改一個簡單文字檔。
- 可以在修改重要設定檔前先備份。

### 為什麼需要這個概念

Linux 錯誤訊息通常藏在 log，服務設定通常藏在 `/etc`。如果不會看檔案內容，就像工程師有監控卻不會讀儀表板。

### 核心重點

- `cat` 適合短檔案，`less` 適合長檔案。
- `tail -f` 適合即時追 log。
- `grep` 用關鍵字搜尋。
- vim 至少要會插入模式、一般模式、存檔、離開。

### 指令地圖

| 指令 | 用途 | 會不會修改系統 | 常見使用時機 |
| --- | --- | --- | --- |
| `cat file` | 一次印出短檔案 | 不會 | 看短設定檔或版本資訊 |
| `less file` | 分頁閱讀長檔案 | 不會 | 看長 log 或長設定檔 |
| `head file` | 看檔案開頭 | 不會 | 快速確認檔案格式 |
| `tail file` | 看檔案結尾 | 不會 | 看最新 log |
| `tail -f file` | 持續追蹤新增內容 | 不會 | 觀察服務即時 log |
| `grep KEY file` | 搜尋關鍵字 | 不會 | 從設定或 log 找線索 |
| `vim file` | 編輯檔案 | 會 | 修改設定或建立筆記 |

### 實作任務情境

你接到一台 Rocky Linux 主機，想確認它的版本資訊，並建立一份自己的練習筆記。這會用到「讀檔、搜尋、編輯」三個能力。

### 操作前檢查

- `/etc/os-release` 是系統版本資訊檔，本練習只讀取，不會修改。
- `notes.txt` 會建立在目前目錄；建議先在 `~/linux-practice` 內操作。
- 如果你不熟 vim，先記住三個鍵：`i` 進入插入模式、`Esc` 回一般模式、`:wq` 存檔離開。

### Step-by-step 實作

```bash
# 範例用途：查系統版本檔、搜尋關鍵字，並用 vim 建立筆記。
# 輸入說明：/etc/os-release 是發行版資訊檔；VERSION 是搜尋關鍵字。
# 輸出結果：會顯示系統版本，並建立 notes.txt。
cd "$HOME/linux-practice"
cat /etc/os-release
grep VERSION /etc/os-release
vim notes.txt
cat notes.txt
```

1. 執行 `cd "$HOME/linux-practice"`，進入前一堂建立的練習資料夾。
2. 執行 `cat /etc/os-release`，先看完整版本資訊。你應該會看到 `NAME`、`VERSION`、`ID` 等欄位。
3. 執行 `grep VERSION /etc/os-release`，只抓出包含 `VERSION` 的行。這是在練習「從大量文字中抓重點」。
4. 執行 `vim notes.txt`，建立或開啟筆記檔。
5. 按 `i` 進入插入模式，輸入你查到的 Rocky Linux 版本。
6. 按 `Esc` 回一般模式。
7. 輸入 `:wq` 存檔離開。
8. 執行 `cat notes.txt`，確認剛剛輸入的內容真的存進檔案。

### 如果結果和預期不同

- `cd "$HOME/linux-practice"` 失敗：回第 2 堂先建立練習資料夾，或改在 `cd "$HOME"` 後操作。
- 進 vim 後不知道怎麼輸入：先按 `i`，左下角通常會出現 `-- INSERT --`。
- 不知道怎麼離開 vim：按 `Esc`，輸入 `:q!` 可以不存檔離開。
- `grep VERSION /etc/os-release` 沒輸出：確認 `VERSION` 是否拼錯，也可以先用 `cat` 看檔案內容。

### 做完後檢查

```bash
pwd
ls -l notes.txt
cat notes.txt
```

你應該能確認 `notes.txt` 存在，並且內容是你剛才寫入的筆記。

### 負面例子 / 錯誤用法

錯誤做法：直接編輯重要設定檔，沒有備份。

問題：設定檔一旦改壞，服務可能起不來；如果沒有備份，你很難快速回復。

修正方向：

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak.$(date +%F)
sudo vim /etc/ssh/sshd_config
```

先備份，再修改。正式環境還要知道怎麼驗證設定檔語法與回復。

### 小練習

1. 用 `grep NAME /etc/os-release` 找出發行版名稱。
2. 用 vim 在 `notes.txt` 追加一行：`I can edit files with vim.`
3. 用 `cat notes.txt` 驗證內容。
4. 練習用 `:q!` 開檔後不存檔離開，確認它和 `:wq` 的差異。

### Junior 常見誤解

- 打開 vim 不能輸入，不是壞掉，而是還在一般模式。
- `tail -f` 會持續追蹤檔案，要按 `Ctrl+C` 停止。
- `/etc` 多是系統設定，修改前要備份。

### 一句話總結

會看、會找、會改文字檔，是 Linux 排錯與維運的基本生存能力。

---

## 第 4 堂：Linux 基礎檔案權限與基礎帳號管理

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit04.php

### 這堂課主要在講什麼

Linux 權限的核心是「誰可以對哪個檔案做什麼事」。每個檔案通常有 owner、group、others 三種身份，每種身份有 read、write、execute 三種權限。

### 學完這堂你應該會做到什麼

- 可以看懂 `ls -l` 的權限欄位。
- 可以分辨檔案和目錄的 `x` 權限差異。
- 可以用 `chmod` 調整基本權限。
- 可以知道為什麼 `chmod 777` 不是好解法。

### 為什麼需要這個概念

權限設太鬆會造成資料外洩或服務被竄改；設太緊又會讓程式無法讀設定、寫 log、執行檔案。你需要看懂 `ls -l` 的權限欄。

### 核心重點

- `r` 是讀取，`w` 是寫入，`x` 是執行或進入目錄。
- 檔案的 `x` 代表可執行；目錄的 `x` 代表可進入。
- `chmod` 改權限，`chown` 改擁有者，`chgrp` 改群組。
- 不要把 `chmod 777` 當萬用解法。

### 指令地圖

| 指令 | 用途 | 會不會修改系統 | Junior 要注意什麼 |
| --- | --- | --- | --- |
| `ls -l` | 查看權限、擁有者、群組 | 不會 | 先看現況再改 |
| `chmod` | 修改權限 | 會 | 改太鬆會有安全風險 |
| `chown` | 修改擁有者 | 會 | 常需要 `sudo`，正式機要很小心 |
| `id` | 查看使用者與群組 | 不會 | 權限問題常和群組有關 |

### 實作任務情境

你要建立一個簡單腳本，讓自己可以執行，其他人可以讀與執行，但不能修改。這是部署腳本、工具腳本很常見的權限需求。

### 操作前檢查

- 請在 `~/linux-practice` 練習，不要在 `/usr/bin` 或 `/etc` 建檔。
- 本練習會修改練習檔案權限，但不會動系統檔案。
- 如果你看到 `Permission denied`，先查 `pwd` 和 `ls -l`，不要直接 `sudo chmod 777`。

### Step-by-step 實作

```bash
# 範例用途：建立腳本，調整成只有擁有者可改、所有人可讀與執行。
# 輸入說明：755 代表 owner=rwx, group=rx, others=rx。
# 輸出結果：hello.sh 可以被執行，但只有 owner 能修改。
cd "$HOME/linux-practice"
printf '#!/usr/bin/env bash\necho "hello permission"\n' > hello.sh
ls -l hello.sh
chmod 755 hello.sh
ls -l hello.sh
./hello.sh
```

1. 建立 `hello.sh`，內容是一個會印出文字的小腳本。
2. 先用 `ls -l hello.sh` 看原本權限，通常一開始沒有執行權限。
3. 執行 `chmod 755 hello.sh`，讓 owner 可讀寫執行，group 和 others 可讀執行。
4. 再用 `ls -l hello.sh` 驗證權限是否變成類似 `-rwxr-xr-x`。
5. 執行 `./hello.sh`，確認腳本能跑。

### 如果結果和預期不同

- `./hello.sh` 顯示 `Permission denied`：代表沒有執行權限，檢查 `ls -l`。
- `command not found`：你可能沒有加 `./`，目前目錄通常不在 `PATH` 裡。
- 權限看不懂：把 `rwxr-xr-x` 拆成三段：owner、group、others。

### 做完後檢查

```bash
ls -l hello.sh
id
```

你應該能說出目前使用者是誰、屬於哪些群組，以及 `hello.sh` 對不同身份開了哪些權限。

### 負面例子 / 錯誤用法

錯誤做法：

```bash
sudo chmod -R 777 /var/www
```

問題：這會讓太多人可以寫入網站目錄，可能造成檔案被竄改或惡意程式被放進去。

修正方向：先確認服務使用者、檔案擁有者和真正需要寫入的目錄，只針對 upload/cache 這類目錄開必要權限。

### 小練習

1. 執行 `chmod 644 hello.sh`，再試 `./hello.sh`，觀察錯誤。
2. 改回 `chmod 755 hello.sh`，確認可以再次執行。
3. 用自己的話解釋 `755` 和 `644` 的差異。

### Junior 常見誤解

- `777` 不是「修好權限」，只是把門全打開。
- 目錄沒有 `x` 時，即使有 `r` 也不一定能進去查看內容。
- 使用者屬於哪些群組會影響實際權限，可用 `id` 查看。

### 一句話總結

Linux 權限不是背數字，而是知道誰需要做什麼，然後只開必要的門。

---

## 第 5 堂：權限應用、程序之觀察與基本管理

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit05.php

### 這堂課主要在講什麼

程序是正在執行中的程式。你打開 shell、執行 `sleep`、啟動 nginx，背後都會有 process。系統管理時，你需要會看哪些程序正在跑、耗了多少資源，以及如何安全停止它。

### 學完這堂你應該會做到什麼

- 可以用 `ps` 和 `top` 觀察程序。
- 可以理解 PID 是程序編號。
- 可以把命令放到背景執行並用 `jobs` 查看。
- 可以先用一般 `kill` 結束程序，而不是一開始就 `kill -9`。

### 為什麼需要這個概念

正式主機常見問題是 CPU 滿載、記憶體爆掉、服務卡住。你如果只會重開機，會看不到真正原因。

### 核心重點

- `ps aux` 看程序快照。
- `top` 或 `htop` 看即時資源。
- `kill` 送 signal，不是永遠等於強制刪除。
- `&` 可讓命令在背景執行。

### 指令地圖

| 指令 | 用途 | 會不會修改系統 | Junior 要注意什麼 |
| --- | --- | --- | --- |
| `ps aux` | 看程序快照 | 不會 | 適合搜尋某個 process |
| `top` | 看即時資源 | 不會 | 按 `q` 離開 |
| `jobs` | 看目前 shell 的背景工作 | 不會 | 只看目前 shell 啟動的 job |
| `kill PID` | 要求程序結束 | 會 | 先用一般 kill |
| `kill -9 PID` | 強制結束 | 會 | 最後手段，可能造成資料未收尾 |

### 實作任務情境

你啟動了一個背景工作，但後來想找到它並結束它。這是在正式主機排查卡住程序時的基本流程，只是這裡用安全的 `sleep` 來練。

### 操作前檢查

- 這個練習只會建立一個 `sleep` 背景程序。
- 不要對不認識的 PID 使用 `kill`。
- 結束程序前先確認 PID 是你剛剛建立的那個。

### Step-by-step 實作

```bash
# 範例用途：建立一個背景 sleep 程序，查到它，再結束它。
# 輸入說明：sleep 300 會等待 300 秒；& 讓它在背景執行。
# 輸出結果：jobs 會看到背景工作，kill 會結束該程序。
sleep 300 &
jobs
ps aux | grep '[s]leep 300'
kill %1
jobs
```

1. 執行 `sleep 300 &`，建立一個背景工作。
2. 執行 `jobs`，確認目前 shell 有背景工作。
3. 執行 `ps aux | grep '[s]leep 300'`，從所有程序中找出 `sleep`。
4. 執行 `kill %1`，結束 job 編號 1。
5. 再執行 `jobs`，確認背景工作已結束。

### 如果結果和預期不同

- `jobs` 沒看到東西：可能 `sleep` 已經結束，重新執行 `sleep 300 &`。
- `kill %1` 失敗：job 編號可能不是 1，先看 `jobs` 的編號。
- `ps aux` 看到很多行：先找 command 欄位是 `sleep 300` 的那一行。

### 做完後檢查

```bash
jobs
ps aux | grep '[s]leep 300'
```

兩個命令都不應該再看到剛才的 `sleep 300`。

### 負面例子 / 錯誤用法

錯誤做法：

```bash
sudo kill -9 1234
```

問題：`-9` 是強制終止，程式沒有機會收尾，可能造成暫存檔、lock、資料寫入中斷。

修正方向：先用一般 `kill PID`，再查 log；只有程序完全不回應時才考慮 `kill -9`。

### 小練習

1. 開兩個 `sleep 300 &`，用 `jobs` 看它們的 job 編號。
2. 分別用 `kill %1`、`kill %2` 結束。
3. 用 `top` 觀察系統，再按 `q` 離開。

### Junior 常見誤解

- PID 不是固定的；每次程序啟動都可能不同。
- `kill` 是送 signal，不是「刪檔」。
- `root` 可以看到與管理更多程序，但也更容易誤殺重要服務。

### 一句話總結

看懂 process，才有能力判斷系統是真的壞了，還是某個程序卡住了。

---

## 第 6 堂：基礎檔案系統管理

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit06.php

### 這堂課主要在講什麼

檔案系統是磁碟能被作業系統使用的方式。Linux 把磁碟、分割區、檔案系統和掛載點串起來，最後你才會看到 `/home`、`/var`、`/data` 這些目錄。

### 學完這堂你應該會做到什麼

- 可以用 `lsblk` 看磁碟與分割區。
- 可以用 `df -h` 看已掛載檔案系統容量。
- 可以用 `du -sh` 看目錄用量。
- 可以知道 `mkfs` 會格式化，不是查詢命令。

### 為什麼需要這個概念

空間不足、磁碟掛不上、開機進 emergency mode，通常都和檔案系統或 `/etc/fstab` 有關。這是伺服器維運很常見的底層問題。

### 核心重點

- `lsblk` 看磁碟與分割區。
- `df -h` 看已掛載檔案系統使用量。
- `du -sh` 看目錄占用。
- `/etc/fstab` 控制開機自動掛載，寫錯可能導致開機失敗。

### 指令地圖

| 指令 | 用途 | 會不會修改系統 | Junior 要注意什麼 |
| --- | --- | --- | --- |
| `lsblk` | 查看 block device | 不會 | 先看磁碟、分割區、掛載點 |
| `lsblk -f` | 查看檔案系統與 UUID | 不會 | 修改 fstab 前常用 |
| `df -h` | 查看檔案系統容量 | 不會 | 看哪個掛載點滿了 |
| `du -sh DIR` | 查看目錄用量 | 不會 | 找出大目錄 |
| `mkfs.xfs` | 建立檔案系統 | 會，會清掉資料 | 只能對確認好的新分割區使用 |

### 實作任務情境

你收到告警說主機空間快滿了。你要先知道有哪些磁碟、哪些掛載點、哪個目錄占空間。這堂先做只讀查詢，不急著分割或格式化。

### 操作前檢查

- 這堂主要使用查詢命令，不會修改磁碟。
- 看到 `mkfs`、`fdisk`、`parted` 先停下來，確認是否在測試 VM。
- 練習進階磁碟前，先拍 VM snapshot。

### Step-by-step 實作

```bash
# 範例用途：查看磁碟結構與目前空間使用量。
# 輸入說明：不修改磁碟，只讀取狀態。
# 輸出結果：會看到磁碟、掛載點、容量與使用率。
lsblk
lsblk -f
df -h
du -sh "$HOME"
```

1. 執行 `lsblk`，看磁碟、分割區與掛載點。
2. 執行 `lsblk -f`，看檔案系統類型與 UUID。
3. 執行 `df -h`，看每個掛載點的容量與使用率。
4. 執行 `du -sh "$HOME"`，看自己家目錄占多少空間。

進階練習只能在測試 VM 的新增磁碟上做：

```bash
# 危險提醒：mkfs 會格式化目標裝置，請勿對系統碟操作。
# /dev/sdb1 只是範例，實際要用 lsblk 確認。
sudo mkfs.xfs /dev/sdb1
sudo mkdir -p /data
sudo mount /dev/sdb1 /data
df -h /data
```

### 如果結果和預期不同

- `lsblk` 沒看到新磁碟：可能 VM 沒加磁碟，或系統尚未重新掃描。
- `df -h` 空間滿了：不要立刻刪檔，先用 `du` 找大目錄。
- `mount` 失敗：確認裝置名稱、檔案系統、掛載點是否存在。

### 做完後檢查

你應該能指出：

- 哪個是整顆磁碟，例如 `/dev/sda`。
- 哪個是分割區，例如 `/dev/sda1`。
- 哪個目錄是掛載點，例如 `/`、`/boot`、`/data`。

### 負面例子 / 錯誤用法

錯誤做法：看到 `/dev/sda` 就直接 `mkfs.xfs /dev/sda`。

問題：`/dev/sda` 很可能是系統碟，格式化後系統資料會消失。

修正方向：先 `lsblk -f`、確認磁碟大小、掛載點、UUID，再操作未使用的新磁碟。

### 小練習

1. 找出目前 `/` 掛載點的使用率。
2. 找出自己家目錄占用大小。
3. 用自己的話解釋 `df` 和 `du` 差在哪裡。

### Junior 常見誤解

- `/dev/sdb` 是整顆磁碟，`/dev/sdb1` 才是常見分割區。
- `mount` 只是掛載到目前系統，重開機不一定還在；永久掛載要處理 `/etc/fstab`。
- `df` 看檔案系統剩餘空間，`du` 看目錄占用。

### 一句話總結

磁碟操作前先看清楚裝置、分割區、檔案系統與掛載點，因為格式化沒有後悔鍵。

---

## 第 7 堂：認識 bash 基礎與系統救援

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit07.php

### 這堂課主要在講什麼

bash 不只是打指令的地方，它有變數、環境變數、alias、命令替換、歷史紀錄。系統救援則是當開機或設定壞掉時，進入救援模式修復。

### 學完這堂你應該會做到什麼

- 可以查看與理解 `$PATH`。
- 可以建立暫時 alias。
- 可以知道 `.bashrc` 修改後要重新載入。
- 可以說出救援模式通常用來修什麼問題。

### 為什麼需要這個概念

很多工具依賴環境變數，例如 `PATH` 決定 shell 去哪裡找命令。救援能力則讓你在主機不能正常開機時仍有修復方向。

### 核心重點

- `echo $PATH` 看命令搜尋路徑。
- `alias` 可建立短命令，但不要讓它掩蓋危險操作。
- `.bashrc` 是互動 shell 常見設定檔。
- 救援模式通常用於修 `/etc/fstab`、root 密碼、boot loader、檔案系統問題。

### 指令地圖

| 指令 | 用途 | 會不會修改系統 | Junior 要注意什麼 |
| --- | --- | --- | --- |
| `echo $PATH` | 看命令搜尋路徑 | 不會 | PATH 壞掉會找不到命令 |
| `alias` | 建立命令別名 | 會影響目前 shell | 初學不要蓋掉危險命令行為 |
| `type COMMAND` | 看命令來源 | 不會 | 判斷是 alias、shell builtin 還是外部命令 |
| `source ~/.bashrc` | 重新載入設定 | 會影響目前 shell | 改壞可能影響命令環境 |

### 實作任務情境

你想讓常用的 `ls -alF` 變成短命令 `ll`，並觀察 shell 是怎麼找命令的。這是理解 shell 設定檔的第一步。

### 操作前檢查

- 本練習只建立暫時 alias，不會永久修改 `.bashrc`。
- 不要建立會讓危險命令變得更危險的 alias。
- 如果 shell 行為怪怪的，開新終端機通常會回到乾淨狀態。

### Step-by-step 實作

```bash
# 範例用途：觀察 bash 環境與建立暫時 alias。
# 輸入說明：alias 只影響目前 shell；重開終端機通常會消失。
# 輸出結果：ll 會變成 ls -alF 的簡寫。
echo "$SHELL"
echo "$PATH"
type ls
alias ll='ls -alF'
type ll
ll
```

1. 執行 `echo "$SHELL"`，確認目前使用的 shell。
2. 執行 `echo "$PATH"`，看 shell 搜尋命令的目錄順序。
3. 執行 `type ls`，看 `ls` 是什麼類型的命令。
4. 建立 `alias ll='ls -alF'`。
5. 執行 `type ll`，確認 `ll` 是 alias。
6. 執行 `ll`，看詳細目錄列表。

### 如果結果和預期不同

- `ll` 顯示 command not found：確認 alias 是否在同一個 shell 裡建立。
- `type ll` 沒有顯示 alias：可能你開了新 shell，alias 沒有被保留。
- PATH 看起來被改壞：開新終端機，或用完整路徑執行命令，例如 `/usr/bin/ls`。

### 做完後檢查

```bash
alias
type ll
```

你應該能看到 `ll` 對應到 `ls -alF`。

### 負面例子 / 錯誤用法

錯誤做法：

```bash
alias rm='rm -rf'
```

問題：這會把刪除命令變得更危險，一打錯就可能遞迴刪掉大量資料。

修正方向：可以設定 `alias rm='rm -i'` 作為初學保護，但更重要的是養成先確認路徑的習慣。

### 小練習

1. 建立 `alias lh='ls -lh'`，再用 `type lh` 驗證。
2. 關掉終端機重新開啟，確認暫時 alias 是否消失。
3. 查 `echo "$PATH"`，找出 `/usr/bin` 是否在其中。

### Junior 常見誤解

- 修改 `.bashrc` 後要重新開 shell 或 `source ~/.bashrc` 才生效。
- `PATH` 壞掉時，不是命令不存在，而是 shell 找不到。
- 救援模式不是日常操作模式，是系統壞掉時的修復入口。

### 一句話總結

bash 是你和系統互動的工作環境，懂它才能知道命令為什麼找得到、為什麼會那樣執行。

---

## 第 8 堂：bash 指令連續下達與資料流重導向

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit08.php

### 這堂課主要在講什麼

Linux 命令可以像積木一樣串起來。pipe `|` 把前一個命令的輸出交給下一個命令；redirect `>`、`>>` 把輸出寫到檔案。

### 學完這堂你應該會做到什麼

- 可以分辨 stdout 和 stderr 的概念。
- 可以用 `|` 串接命令。
- 可以用 `>`、`>>`、`2>` 控制輸出。
- 可以用 `tee` 同時顯示與寫檔。

### 為什麼需要這個概念

單一命令只能做一件事；pipe 和 redirect 讓你把查詢、篩選、保存結果串成小型工具。這是 shell script 和自動化的前置能力。

### 核心重點

- `>` 覆蓋檔案，`>>` 追加檔案。
- `2>` 重導錯誤輸出。
- `|` 串接 stdout 到下一個命令。
- `tee` 可同時顯示並寫入檔案。

### 指令地圖

| 符號 / 指令 | 用途 | 會不會修改系統 | Junior 要注意什麼 |
| --- | --- | --- | --- |
| `|` | 把前一個命令輸出交給下一個 | 不直接修改 | 傳的是文字輸出 |
| `>` | 覆蓋寫入檔案 | 會 | 可能清掉原檔案 |
| `>>` | 追加寫入檔案 | 會 | 適合保留原內容 |
| `2>` | 寫入錯誤輸出 | 會 | stderr 和 stdout 不同 |
| `tee` | 顯示並寫檔 | 會 | 常用來保留查詢結果 |

### 實作任務情境

你要列出 `/etc` 底下跟 ssh 有關的項目，並把結果存成報告。這很像在排查設定或整理交接資料。

### 操作前檢查

- 本練習只查詢 `/etc` 檔名，不修改系統設定。
- 報告檔會建立在目前目錄，建議在 `~/linux-practice` 執行。
- 注意 `>` 會覆蓋同名檔案。

### Step-by-step 實作

```bash
# 範例用途：列出 /etc 中包含 ssh 的檔名，存成報告。
# 輸入說明：grep -i 忽略大小寫；tee 同時顯示並寫檔。
# 輸出結果：ssh-files.txt 會保存查詢結果。
cd "$HOME/linux-practice"
find /etc -maxdepth 2 -iname '*ssh*' 2>/dev/null | tee ssh-files.txt
cat ssh-files.txt
```

1. 進入練習目錄。
2. 用 `find` 搜尋 `/etc` 底下兩層以內名稱含 ssh 的項目。
3. 用 `2>/dev/null` 把沒有權限讀取的錯誤訊息丟掉，避免干擾結果。
4. 用 `tee ssh-files.txt` 一邊顯示結果，一邊寫入檔案。
5. 用 `cat ssh-files.txt` 驗證報告內容。

### 如果結果和預期不同

- 畫面出現很多 `Permission denied`：確認是否少了 `2>/dev/null`。
- `ssh-files.txt` 是空的：確認命令是否打錯，也可以把 `-maxdepth 2` 放寬。
- 結果被覆蓋：使用 `tee -a` 或 `>>` 追加，而不是覆蓋。

### 做完後檢查

```bash
ls -l ssh-files.txt
wc -l ssh-files.txt
```

你應該能看到報告檔存在，並知道裡面有幾行結果。

### 負面例子 / 錯誤用法

錯誤做法：

```bash
echo "new config" > important.conf
```

問題：`>` 會覆蓋原檔案，重要資料可能直接消失。

修正方向：不確定時先備份檔案，或使用 `>>` 追加；要寫設定檔前先確認檔案內容。

### 小練習

1. 把 `cat /etc/os-release` 的結果存成 `os-release.txt`。
2. 用 `grep VERSION os-release.txt` 搜尋版本行。
3. 試著用 `>>` 追加一行自己的註解。

### Junior 常見誤解

- pipe 傳的是文字輸出，不是把整個程式「接在一起」。
- `>` 和 `>>` 差一個字元但後果差很多。
- 錯誤訊息預設走 stderr，不一定會被 `>` 收進檔案。

### 一句話總結

pipe 和 redirect 讓你把小命令組合成可重複、可保存、可排查的工作流程。

---

## 第 9 堂：正規表示法與 shell script 初探

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit09.php

### 這堂課主要在講什麼

這堂課開始把「一次打一個命令」變成「寫成可重複執行的小工具」。regex 用來描述文字模式，shell script 用來固定查詢、判斷、輸出的流程。

### 學完這堂你應該會做到什麼

- 可以用 `grep` 搜尋簡單模式。
- 可以理解 `^`、`$`、`.` 這幾個基本 regex 符號。
- 可以寫一個簡單 bash script。
- 可以知道變數引用要加雙引號。

### 為什麼需要這個概念

你不可能每天手動查 100 個 log。會 regex 與 script 後，你可以把「查詢、篩選、判斷、輸出」固定成工具。

### 核心重點

- `grep` 搜尋文字模式。
- `^` 表示行首，`$` 表示行尾，`.` 表示任一字元。
- script 第一行常見 `#!/usr/bin/env bash`。
- 變數引用要加雙引號，減少空白與特殊字元造成的錯誤。

### 指令地圖

| 指令 / 語法 | 用途 | 會不會修改系統 | Junior 要注意什麼 |
| --- | --- | --- | --- |
| `grep PATTERN file` | 搜尋文字模式 | 不會 | pattern 不是普通文字時要懂 regex |
| `chmod +x script.sh` | 讓 script 可執行 | 會 | 只對自己的 script 操作 |
| `"$VAR"` | 安全引用變數 | 不會 | 避免空白與空字串造成危險 |
| `if ... then ... fi` | 條件判斷 | 不直接修改 | script 的核心控制流程 |

### 實作任務情境

你要寫一個小 script，檢查根目錄 `/` 使用率是否超過門檻。這不是完整監控系統，但能練到變數、命令替換、條件判斷。

### 操作前檢查

- 在 `~/linux-practice` 裡建立 script。
- script 只讀取 `df` 結果，不會修改系統。
- 門檻值先用 80，可以自行調整測試。

### Step-by-step 實作

```bash
# 範例用途：建立一個檢查磁碟使用率的簡單 script。
# 輸入說明：THRESHOLD 是警戒百分比，不含 % 符號。
# 輸出結果：列出根目錄使用率，超過門檻時顯示 WARNING。
cd "$HOME/linux-practice"
cat > check-disk.sh <<'EOF'
#!/usr/bin/env bash
set -u

THRESHOLD=80
USAGE=$(df -P / | awk 'NR==2 { gsub("%", "", $5); print $5 }')

echo "Root filesystem usage: ${USAGE}%"

if [ "$USAGE" -ge "$THRESHOLD" ]; then
  echo "WARNING: disk usage is higher than ${THRESHOLD}%"
else
  echo "OK: disk usage is under ${THRESHOLD}%"
fi
EOF

chmod +x check-disk.sh
./check-disk.sh
```

1. 用 heredoc 建立 `check-disk.sh`。
2. `df -P /` 查根目錄容量，`awk` 抓第二行第五欄使用率。
3. `gsub("%", "", $5)` 把 `%` 拿掉，讓 bash 可以做數字比較。
4. `chmod +x` 讓 script 可執行。
5. 執行 `./check-disk.sh` 看結果。

### 如果結果和預期不同

- `Permission denied`：忘了執行 `chmod +x check-disk.sh`。
- `integer expression expected`：`USAGE` 不是純數字，檢查 `df` / `awk` 輸出。
- `command not found`：執行目前目錄 script 要用 `./check-disk.sh`。

### 做完後檢查

```bash
ls -l check-disk.sh
bash -n check-disk.sh
./check-disk.sh
```

`bash -n` 沒輸出通常代表語法檢查通過。

### 負面例子 / 錯誤用法

錯誤做法：

```bash
rm -rf $TARGET/*
```

問題：如果 `TARGET` 是空字串，命令可能變成危險刪除；如果路徑有空白，也可能被拆成多段。

修正方向：

```bash
if [ -n "${TARGET:-}" ] && [ -d "$TARGET" ]; then
  rm -rf "$TARGET"/*
fi
```

先檢查變數是否有值、目錄是否存在，再用雙引號包住變數。

### 小練習

1. 把 `THRESHOLD` 改成 1，觀察是否出現 WARNING。
2. 用 `grep '^NAME=' /etc/os-release` 練習行首搜尋。
3. 用自己的話說明 `.` 在 regex 裡為什麼不是小數點。

### Junior 常見誤解

- shell script 不是只把命令貼在一起；要處理輸入、錯誤與邊界條件。
- regex 的 `.` 不是小數點，而是任一字元；要匹配真正小數點常用 `\.`。
- 變數不加雙引號，在路徑有空白時容易出錯。

### 一句話總結

script 的價值不是炫技，而是把你確認過的排查流程變成可重複使用的小工具。

---

## 第 10 堂：使用者管理與 ACL 權限設定

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit10.php

### 這堂課主要在講什麼

Linux 是多使用者系統。你需要會建立帳號、設定密碼、加入群組、管理 sudo 權限，並在傳統 owner/group/others 不夠用時使用 ACL。

### 學完這堂你應該會做到什麼

- 可以建立測試使用者。
- 可以查看使用者所屬群組。
- 可以理解 `sudo` 和 `su` 差異。
- 可以用 ACL 對單一使用者加權限。

### 為什麼需要這個概念

正式主機通常不是一個人使用。帳號分離可以追蹤操作、限制權限、降低共用密碼的風險。ACL 則可以在不改 owner 的情況下做細部授權。

### 核心重點

- `useradd` 建立使用者，`passwd` 設密碼。
- `usermod -aG wheel user` 在 RHEL 系常用來給 sudo 權限。
- `sudo` 是授權執行，不是切換人格無痕操作。
- ACL 可用 `setfacl` / `getfacl` 做細部權限。

### 指令地圖

| 指令 | 用途 | 會不會修改系統 | Junior 要注意什麼 |
| --- | --- | --- | --- |
| `id USER` | 查看使用者與群組 | 不會 | 先查再改 |
| `sudo useradd USER` | 建立使用者 | 會 | 正式環境要符合命名規範 |
| `sudo passwd USER` | 設定密碼 | 會 | 不要用弱密碼 |
| `sudo usermod -aG wheel USER` | 加入 wheel 群組 | 會 | `-aG` 不要漏 `a` |
| `setfacl` | 設定 ACL | 會 | 設完要用 `getfacl` 驗證 |

### 實作任務情境

你要建立一個開發者帳號 `dev01`，讓他可以依授權使用 sudo，並讓他對某個共享檔案有讀寫權限。

### 操作前檢查

- 這堂會修改系統帳號，請只在練習 VM 執行。
- 確認你有 sudo 權限。
- 如果 VM 是一次性練習環境，做完可以刪除測試帳號或重建 VM。

### Step-by-step 實作

```bash
# 範例用途：建立一個開發者帳號並加入 wheel 群組。
# 輸入說明：dev01 是範例帳號；wheel 在 Rocky/RHEL 系通常代表可 sudo。
# 輸出結果：dev01 可在授權後使用 sudo。
sudo useradd dev01
sudo passwd dev01
sudo usermod -aG wheel dev01
id dev01
```

ACL 練習：

```bash
# 範例用途：讓 dev01 對 shared.txt 有讀寫權限，不改檔案 owner。
cd "$HOME/linux-practice"
touch shared.txt
setfacl -m u:dev01:rw shared.txt
getfacl shared.txt
```

1. 建立 `dev01` 帳號。
2. 設定密碼。
3. 把 `dev01` 加入 `wheel` 群組。
4. 用 `id dev01` 驗證群組。
5. 建立 `shared.txt`，用 ACL 給 `dev01` 讀寫權限。
6. 用 `getfacl` 驗證 ACL。

### 如果結果和預期不同

- `useradd: user already exists`：帳號已存在，先用 `id dev01` 查。
- `dev01` 無法 sudo：確認是否已加入 `wheel`，也要重新登入讓群組生效。
- `setfacl: command not found`：可能缺少 ACL 工具，Rocky 通常可用 `sudo dnf install acl` 安裝。

### 做完後檢查

```bash
id dev01
getfacl "$HOME/linux-practice/shared.txt"
```

你應該看到 `dev01` 在群組資訊中，以及 `shared.txt` 有 `user:dev01:rw-` 類似 ACL。

### 負面例子 / 錯誤用法

錯誤做法：多人共用同一個 `admin` 帳號。

問題：出事時不知道是誰操作，也很難做到最小權限與離職停權。

修正方向：每個人用自己的帳號，透過群組、sudo 規則與 ACL 管理權限。

### 小練習

1. 建立另一個測試帳號 `dev02`。
2. 不把 `dev02` 加入 wheel，觀察它和 `dev01` 的差異。
3. 用 `getfacl` 說出 `shared.txt` 對 owner、group、dev01 各有什麼權限。

### Junior 常見誤解

- `su` 和 `sudo` 不同；`su` 是切換使用者，`sudo` 是用授權執行命令。
- 把人加進 `wheel` 前要確認公司規範，不是所有人都該有 sudo。
- ACL 是補充，不是取代基本權限觀念。

### 一句話總結

帳號與權限管理的目標不是方便，而是讓每個人只拿到完成工作需要的權限。

---

## 第 11 堂：基礎設定、備份、檔案壓縮打包與工作排程

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit11.php

### 這堂課主要在講什麼

這堂課處理日常維運常見的三件事：系統時間、備份壓縮、排程。它們看似雜，但都和「系統能不能穩定、可追蹤、可復原」有關。

### 學完這堂你應該會做到什麼

- 可以查看系統時間與時區。
- 可以用 `tar` 打包壓縮資料夾。
- 可以測試備份檔是否能還原。
- 可以理解 cron 的基本格式。

### 為什麼需要這個概念

時間錯會讓 log 對不起來、憑證驗證失敗、排程在錯誤時間跑。備份不會做，出事時就只能祈禱。排程不會設，例行工作只能靠人手。

### 核心重點

- `timedatectl` 管理時間與時區。
- `tar` 打包，`gzip` / `xz` 壓縮。
- `cron` 適合傳統週期任務，systemd timer 是現代替代方案之一。
- 備份一定要測試還原。

### 指令地圖

| 指令 | 用途 | 會不會修改系統 | Junior 要注意什麼 |
| --- | --- | --- | --- |
| `timedatectl` | 查看時間設定 | 不會 | 時區影響 log 與排程 |
| `timedatectl set-timezone` | 設定時區 | 會 | 正式環境需遵守團隊標準 |
| `tar -czf` | 建立 gzip 壓縮包 | 會建立檔案 | 備份前確認路徑 |
| `tar -tzf` | 列出壓縮包內容 | 不會 | 還原前先看內容 |
| `crontab -e` | 編輯個人排程 | 會 | 先在測試環境練 |

### 實作任務情境

你要確認 VM 時區，並把 `linux-practice` 練習目錄打包成備份檔，然後檢查備份內容是否正確。

### 操作前檢查

- 本練習會建立壓縮檔，不會刪資料。
- 設定時區需要 sudo；若只是學習，可先只查詢。
- 排程範例先閱讀，不要直接丟正式環境。

### Step-by-step 實作

```bash
# 範例用途：查看時間設定，備份練習目錄。
# 輸入說明：Asia/Taipei 是台灣常用時區。
# 輸出結果：產生 linux-practice-backup.tar.gz。
timedatectl
sudo timedatectl set-timezone Asia/Taipei
cd "$HOME"
tar -czf linux-practice-backup.tar.gz linux-practice
tar -tzf linux-practice-backup.tar.gz | head
```

cron 範例：

```cron
# 每天 02:30 執行備份腳本的 crontab 範例，不要直接照貼到正式環境。
30 2 * * * /home/student/backup.sh
```

1. 用 `timedatectl` 查看目前時間、時區與 NTP 狀態。
2. 用 `timedatectl set-timezone` 設定時區。
3. 回到家目錄，使用 `tar -czf` 備份 `linux-practice`。
4. 用 `tar -tzf` 查看壓縮包內容，確認真的備到目標資料夾。
5. 閱讀 cron 格式：分、時、日、月、星期、命令。

### 如果結果和預期不同

- `timedatectl set-timezone` 顯示權限不足：需要 sudo。
- `tar: linux-practice: Cannot stat`：代表目錄不存在或不在目前路徑。
- cron 沒執行：先查路徑是否寫絕對路徑、script 是否可執行、環境變數是否不同。

### 做完後檢查

```bash
ls -lh "$HOME/linux-practice-backup.tar.gz"
tar -tzf "$HOME/linux-practice-backup.tar.gz" | head
```

你應該看到備份檔存在，而且內容包含 `linux-practice/`。

### 負面例子 / 錯誤用法

錯誤做法：只建立備份檔，從不測試解壓與還原。

問題：等真的出事才發現備份檔壞掉、路徑錯誤或根本沒有備到重要資料。

修正方向：定期測試還原流程，備份不等於有復原能力。

### 小練習

1. 建立 `restore-test` 目錄，把備份解壓到裡面。
2. 比較原本 `linux-practice` 和還原後的內容。
3. 用自己的話解釋 cron 的 `30 2 * * *`。

### Junior 常見誤解

- 備份不是把檔案壓起來就結束，能還原才算。
- cron 的環境變數很少，script 在手動執行正常不代表排程正常。
- 時區錯不一定會讓程式壞掉，但會讓排查變很痛苦。

### 一句話總結

時間、備份、排程是維運基本功，重點是可驗證、可還原、可追蹤。

---

## 第 12 堂：軟體管理與安裝及登錄檔初探

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit12.php

### 這堂課主要在講什麼

CentOS 7 教材會提 `rpm` 與 `yum`；2026 年在 Rocky/RHEL 系建議以 `dnf` 為主，`rpm` 用來查詢底層套件資訊。登錄檔則從傳統 `/var/log` 與 systemd journal 兩邊看。

### 學完這堂你應該會做到什麼

- 可以用 `dnf` 搜尋、安裝、移除套件。
- 可以用 `rpm -q` 查套件是否安裝。
- 可以用 `journalctl` 查 systemd journal。
- 可以知道為什麼不該亂加第三方 repository。

### 為什麼需要這個概念

安裝軟體不是下載執行檔而已。Linux 套件管理會處理依賴、版本、來源與更新。log 則是排查服務失敗的第一手線索。

### 核心重點

- `dnf search` 搜尋套件。
- `dnf install` 安裝，`dnf remove` 移除，`dnf update` 更新。
- `rpm -q` 查套件是否安裝。
- `journalctl` 查 systemd journal。

### 指令地圖

| 指令 | 用途 | 會不會修改系統 | Junior 要注意什麼 |
| --- | --- | --- | --- |
| `dnf search NAME` | 搜尋套件 | 不會 | 先確認套件名稱 |
| `sudo dnf install NAME` | 安裝套件 | 會 | 確認來源與依賴 |
| `sudo dnf remove NAME` | 移除套件 | 會 | 看清楚會移除哪些依賴 |
| `rpm -q NAME` | 查套件是否安裝 | 不會 | 查詢底層安裝狀態 |
| `journalctl` | 查 journal | 不會 | 排錯時很常用 |

### 實作任務情境

你想安裝 `tree` 這個小工具來看目錄樹，安裝後確認它存在，再移除它。這是安全練習套件管理的好目標。

### 操作前檢查

- 需要網路與 sudo 權限。
- 如果是公司正式主機，安裝套件前要確認維運規範。
- 不要任意加入不明 repository。

### Step-by-step 實作

```bash
# 範例用途：安裝 tree，確認來源與版本，再移除。
# 輸入說明：tree 是顯示目錄樹的小工具。
# 輸出結果：tree 可顯示目錄結構；rpm 可查安裝狀態。
dnf search tree
sudo dnf install -y tree
rpm -q tree
tree "$HOME/linux-practice"
sudo dnf remove -y tree
rpm -q tree
```

查 log：

```bash
# 範例用途：查看最近一次開機以來的錯誤等級 log。
journalctl -p err -b
```

1. 搜尋 `tree` 套件。
2. 安裝 `tree`。
3. 用 `rpm -q tree` 驗證已安裝。
4. 用 `tree` 看練習目錄。
5. 移除 `tree`。
6. 再用 `rpm -q tree` 驗證已移除。
7. 用 `journalctl -p err -b` 看本次開機錯誤 log。

### 如果結果和預期不同

- `dnf` 無法連線：確認 DNS、網路、repository 設定。
- `No match for argument`：套件名稱可能錯，先 `dnf search`。
- `journalctl` 太多：加上 `-n 50` 或 `--since today` 縮小範圍。

### 做完後檢查

你應該能說出：

- `dnf` 和 `rpm` 分別適合做什麼。
- 套件安裝前要確認來源。
- 查服務問題時，log 比猜測更可靠。

### 負面例子 / 錯誤用法

錯誤做法：隨便加入不明第三方 repository，再用 root 安裝套件。

問題：可能安裝到不可信套件、覆蓋系統套件版本，造成安全與維護風險。

修正方向：優先使用官方 repository；需要第三方來源時，確認來源、簽章、維護狀態與團隊規範。

### 小練習

1. 用 `dnf info bash` 查 bash 套件資訊。
2. 用 `rpm -q bash` 查是否安裝。
3. 用 `journalctl -n 20` 看最近 20 行 journal。

### Junior 常見誤解

- `dnf` 是高階套件管理工具，`rpm` 是底層套件工具；日常安裝優先用 `dnf`。
- log 不是只有 `/var/log/messages`，systemd journal 也很重要。
- 安裝套件前要在意來源，不只是能不能裝成功。

### 一句話總結

套件管理解決「軟體從哪來、怎麼更新、怎麼移除」，log 解決「出問題時去哪裡找證據」。

---

## 第 13 堂：服務管理與開機流程管理

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit13.php

### 這堂課主要在講什麼

Rocky/RHEL 系主要用 systemd 管理服務。你需要會啟動、停止、重啟、設定開機自動啟動、查看狀態與 log。

### 學完這堂你應該會做到什麼

- 可以用 `systemctl status` 查服務狀態。
- 可以用 `start`、`stop`、`restart`、`reload` 控制服務。
- 可以用 `enable --now` 設定立即啟動且開機自動啟動。
- 可以用 `journalctl -u` 查某個服務的 log。

### 為什麼需要這個概念

後端 API、Web server、資料庫、排程工作都常以服務形式存在。服務掛了，使用者就可能連不上系統；你要知道怎麼查，而不是只會重開機。

### 核心重點

- `systemctl status service` 查狀態。
- `systemctl start/stop/restart/reload` 控制服務。
- `systemctl enable --now` 設定立即啟動且開機自動啟動。
- `journalctl -u service` 查某服務 log。

### 指令地圖

| 指令 | 用途 | 會不會修改系統 | Junior 要注意什麼 |
| --- | --- | --- | --- |
| `systemctl status NAME` | 查服務狀態 | 不會 | 先看 active / failed |
| `sudo systemctl start NAME` | 啟動服務 | 會 | 只影響當下 |
| `sudo systemctl enable NAME` | 設定開機自動 | 會 | 不一定立刻啟動 |
| `sudo systemctl enable --now NAME` | 啟動且開機自動 | 會 | 常用於新服務 |
| `journalctl -u NAME` | 查服務 log | 不會 | 排錯第一站 |

### 實作任務情境

你要在 Rocky Linux VM 安裝 nginx，啟動服務，確認它有在跑，並學會從 log 查問題。

### 操作前檢查

- 需要 sudo 和網路。
- nginx 會開一個 web server 服務；若 VM 有防火牆，外部不一定連得到，但本機可先查服務狀態。
- 正式環境重啟服務前，先確認是否會影響使用者。

### Step-by-step 實作

```bash
# 範例用途：安裝並啟動 nginx，查看服務狀態與 log。
# 輸入說明：nginx 是常見 web server；需要 sudo 權限。
# 輸出結果：nginx 服務啟動，狀態顯示 active。
sudo dnf install -y nginx
sudo systemctl enable --now nginx
systemctl status nginx
journalctl -u nginx -n 30 --no-pager
```

1. 安裝 nginx。
2. 用 `enable --now` 同時啟動 nginx 並設定開機自動啟動。
3. 用 `systemctl status nginx` 看 active 狀態。
4. 用 `journalctl -u nginx` 看服務最近 log。

### 如果結果和預期不同

- `status` 顯示 `failed`：先看 `journalctl -u nginx -n 50`。
- 安裝失敗：查網路與 repository。
- 外部連不到 nginx：可能是防火牆、雲端 security group、nginx listen 設定，不代表服務沒啟動。

### 做完後檢查

```bash
systemctl is-active nginx
systemctl is-enabled nginx
```

預期分別看到 `active` 和 `enabled`。

### 負面例子 / 錯誤用法

錯誤做法：改完設定後直接 `restart`，沒有先檢查設定。

問題：設定檔錯誤會讓服務停掉後起不來。

修正方向：能檢查設定的服務先跑測試，例如 nginx：

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### 小練習

1. 用 `systemctl status nginx` 找出 main PID。
2. 用 `journalctl -u nginx --since today` 看今天的 log。
3. 比較 `enable`、`start`、`enable --now` 的差異。

### Junior 常見誤解

- `start` 只啟動當下，不代表下次開機會自動啟動。
- `enable` 只設定開機自動，不一定立刻啟動；可用 `enable --now`。
- `status` 顯示的最後幾行 log 不一定夠，要搭配 `journalctl`。

### 一句話總結

systemd 是現代 Linux 服務管理核心，會查狀態與 log，比只會重啟重要得多。

---

## 第 14 堂：進階檔案系統管理

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit14.php

### 這堂課主要在講什麼

這堂課處理 RAID、LVM、Quota 等進階儲存主題。初學者不用急著全部實作到熟，但要知道它們分別解決什麼問題。

### 學完這堂你應該會做到什麼

- 可以說出 RAID、LVM、Quota 分別解決什麼問題。
- 可以用 `pvs`、`vgs`、`lvs` 查看 LVM 狀態。
- 可以理解 PV、VG、LV 的分層。
- 可以知道 RAID 不是備份。

### 為什麼需要這個概念

伺服器磁碟常需要擴充、容錯或限制使用量。RAID 處理磁碟容錯，LVM 處理彈性容量管理，Quota 處理使用者空間限制。

### 核心重點

- Software RAID 用 `mdadm` 管理多顆磁碟組合。
- LVM 有 PV、VG、LV 三層。
- Quota 可限制使用者或群組的磁碟用量。
- RAID 不是備份，LVM snapshot 也不是完整備份策略。

### 指令地圖

| 指令 | 用途 | 會不會修改系統 | Junior 要注意什麼 |
| --- | --- | --- | --- |
| `pvs` | 查看 Physical Volume | 不會 | LVM 最底層 |
| `vgs` | 查看 Volume Group | 不會 | 多個 PV 組成的容量池 |
| `lvs` | 查看 Logical Volume | 不會 | 實際給檔案系統使用的邏輯磁碟 |
| `lsblk` | 查看磁碟與掛載 | 不會 | 對照 LVM 與實體磁碟 |
| `mdadm` | 管理 software RAID | 會 | 初學先不要直接操作正式磁碟 |

### 實作任務情境

你接手一台主機，要判斷它有沒有使用 LVM。這堂先做只讀觀察，不直接建立或刪除磁碟配置。

### 操作前檢查

- 本練習只查詢 LVM 狀態，不修改磁碟。
- 如果命令不存在，可能系統沒有安裝 LVM 工具。
- 不要在正式機上嘗試未理解的 RAID / LVM 建立命令。

### Step-by-step 實作

```bash
# 範例用途：只查看目前系統是否使用 LVM，不修改磁碟。
# 輸入說明：pvs/vgs/lvs 分別查看 Physical Volume、Volume Group、Logical Volume。
# 輸出結果：若系統使用 LVM，會看到分層資訊。
sudo pvs
sudo vgs
sudo lvs
lsblk
```

LVM 心智模型：

1. PV：把實體磁碟或分割區交給 LVM 管理。
2. VG：把多個 PV 合成一個容量池。
3. LV：從 VG 切出邏輯磁碟。
4. Filesystem：在 LV 上建立 XFS/ext4。
5. Mount point：掛載到 `/data` 這類目錄使用。

### 如果結果和預期不同

- `pvs` 沒輸出：系統可能沒有使用 LVM，這不一定是錯。
- `command not found`：可能沒有安裝 LVM 工具。
- `lsblk` 看不懂：先找掛載點 `/` 對應到哪個 device。

### 做完後檢查

你應該能畫出這台主機的儲存大概層次：磁碟 / 分割區 / LVM / 檔案系統 / 掛載點。

### 負面例子 / 錯誤用法

錯誤做法：以為 RAID1 就等於備份。

問題：RAID1 只能在單顆磁碟壞掉時維持可用；如果你誤刪資料、被勒索軟體加密、檔案系統損毀，錯誤也會同步到鏡像。

修正方向：RAID 提供可用性，備份提供可復原性，兩者都重要但目的不同。

### 小練習

1. 用 `lsblk` 找出 `/` 掛載點在哪個 device。
2. 如果系統有 LVM，找出 VG 和 LV 名稱。
3. 用自己的話解釋 PV、VG、LV。

### Junior 常見誤解

- LVM 讓容量管理比較彈性，但不是自動變安全。
- Quota 是限制使用量，不是釋放磁碟空間。
- 儲存架構越複雜，越需要文件與備份策略。

### 一句話總結

進階儲存管理是在解決容量、彈性與可用性問題，但它不能取代備份。

---

## 第 15 堂：Linux 系統的準備 Optional

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit15.php

### 這堂課主要在講什麼

這堂課像是一台新 Linux 主機交到你手上後的基線檢查。你要確認版本、更新、時間、帳號、SSH、防火牆、SELinux、log、服務狀態。

### 學完這堂你應該會做到什麼

- 可以用 checklist 檢查新主機基本狀態。
- 可以知道哪些設定和安全有關。
- 可以分辨「先查原因」和「直接關掉安全機制」的差異。
- 可以把檢查結果寫成交接紀錄。

### 為什麼需要這個概念

新主機不是能登入就算完成。基線檢查可以避免時間錯、沒更新、root/SSH 開太鬆、防火牆或 SELinux 被關掉、log 沒地方查這些後續問題。

### 核心重點

- 新主機先確認版本、更新與時間。
- 帳號與 SSH 要避免直接 root 登入與弱密碼。
- 防火牆、SELinux、log、服務狀態都要檢查。
- 不要把關閉安全機制當成排錯第一步。

### 指令地圖

| 指令 | 用途 | 會不會修改系統 | Junior 要注意什麼 |
| --- | --- | --- | --- |
| `cat /etc/os-release` | 查系統版本 | 不會 | 先確認發行版 |
| `sudo dnf update` | 更新套件 | 會 | 正式環境要排維護時間 |
| `timedatectl` | 查時間 | 不會 | 時區與 NTP 很重要 |
| `systemctl status sshd` | 查 SSH | 不會 | 遠端登入核心服務 |
| `firewall-cmd --state` | 查 firewalld | 不會 | 不要隨便關防火牆 |
| `getenforce` | 查 SELinux | 不會 | Enforcing 不等於錯誤 |

### 實作任務情境

你拿到一台剛裝好的 Rocky Linux VM，要做基線檢查，確認它適合拿來後續練習或部署測試服務。

### 操作前檢查

- 這份 checklist 大多是查詢命令。
- `dnf update` 會更新套件，正式環境要先確認維護窗口。
- 請把輸出整理成紀錄，不要只看過就算。

### 完整範圍補強：建立可維護的練習主機

第 15 堂不要只記幾個查詢指令。真正拿到一台新主機時，通常要同時確認「版本、帳號、套件、服務、防火牆、log、備份」這幾個面向，才知道後續練習或部署時問題會從哪裡來。

#### 範例範圍地圖

| 面向 | 會碰到的檔案 / 指令 | 目的 |
| --- | --- | --- |
| 主機資訊 | `/etc/os-release`、`uname -r`、`hostnamectl` | 確認發行版、kernel、主機名稱 |
| 使用者與權限 | `useradd`、`usermod -aG wheel`、`id` | 建立練習帳號並確認 sudo 權限 |
| 套件更新 | `dnf update`、`dnf install` | 讓系統套件維持可維護狀態 |
| 服務管理 | `systemctl status sshd`、`systemctl enable --now` | 確認重要服務有啟動且可開機自動啟動 |
| 防火牆 | `firewall-cmd --list-all` | 確認開放的服務與 port |
| Log 與排查 | `journalctl -xe`、`journalctl -u sshd` | 從錯誤訊息找問題來源 |
| 備份練習 | `tar`、`cron` 或 systemd timer | 建立可還原的練習資料 |

#### Step-by-step 實作：從檢查到留下紀錄

```bash
# 1. 確認這台主機是什麼系統
cat /etc/os-release
uname -r
hostnamectl

# 2. 建立練習帳號，並加入 wheel 群組
sudo useradd student
sudo passwd student
sudo usermod -aG wheel student
id student

# 3. 更新套件並檢查常用服務
sudo dnf update -y
systemctl status sshd --no-pager
systemctl status firewalld --no-pager

# 4. 檢查防火牆目前開放內容
sudo firewall-cmd --list-all

# 5. 建立一份練習資料並打包備份
mkdir -p ~/linux-practice
echo "hello rocky" > ~/linux-practice/readme.txt
tar -czf ~/linux-practice-backup.tar.gz -C ~ linux-practice
ls -lh ~/linux-practice-backup.tar.gz
```

#### 端到端流程

1. 先用版本與主機名稱確認你操作的是正確機器。
2. 建立非 root 的日常操作帳號，避免每一步都直接用 root。
3. 更新套件，讓後續安裝與服務管理建立在穩定基礎上。
4. 檢查 SSH 與 firewalld，確認遠端登入和網路規則可理解。
5. 建立一份小資料並備份，練習「資料在哪裡、怎麼保存、怎麼還原」。

#### 做完後檢查

- `id student` 看得到 `wheel` 群組。
- `systemctl status sshd` 顯示服務正在執行。
- `firewall-cmd --list-all` 能列出目前 zone 的服務與 port。
- `~/linux-practice-backup.tar.gz` 存在，而且大小不是 0。

### Step-by-step 實作：新主機基線 checklist

```bash
# 範例用途：新 Rocky Linux 主機安裝後的基礎檢查。
# 輸入說明：這些命令多數只讀狀態；更新與設定需要 sudo。
# 輸出結果：確認版本、更新、時間、防火牆、SELinux、SSH 服務狀態。
cat /etc/os-release
uname -r
whoami
id
timedatectl
sudo dnf check-update
systemctl status sshd --no-pager
firewall-cmd --state
getenforce
journalctl -p err -b --no-pager
```

1. 查發行版與 kernel。
2. 查目前登入身份與群組。
3. 查時間與時區。
4. 查可更新套件。
5. 查 SSH 服務狀態。
6. 查 firewalld 狀態。
7. 查 SELinux 狀態。
8. 查本次開機錯誤 log。

### 如果結果和預期不同

- `firewall-cmd` 找不到：可能 firewalld 未安裝或未啟動，先確認公司標準。
- `getenforce` 顯示 `Disabled`：確認是否有人關掉 SELinux，以及原因是否有紀錄。
- `journalctl -p err -b` 有錯誤：先記錄錯誤訊息，再查對應服務，不要直接重開機。

### 做完後檢查

請整理一份主機紀錄：

| 項目 | 結果 | 是否需要處理 |
| --- | --- | --- |
| 發行版 | 例如 Rocky Linux 9.x |  |
| Kernel | `uname -r` 輸出 |  |
| 時區 | `timedatectl` 輸出 |  |
| SSH | active / failed |  |
| 防火牆 | running / not running |  |
| SELinux | Enforcing / Permissive / Disabled |  |
| 錯誤 log | 摘要 |  |

### 負面例子 / 錯誤用法

錯誤做法：

```bash
sudo setenforce 0
sudo systemctl stop firewalld
```

問題：這只是把安全檢查拿掉，不是解決問題。正式環境會增加入侵風險，也讓未來問題更難追。

修正方向：先查被擋的是什麼、哪個 port、哪個 SELinux context、哪個服務，再做最小必要調整。

### 小練習

1. 用表格記錄自己的 VM 基線檢查結果。
2. 找出目前 SSH 服務是否 active。
3. 查本次開機是否有 error 等級 log，挑一筆研究它是哪個服務產生。

### Junior 常見誤解

- SELinux 不是錯誤來源的代名詞，它是安全機制。
- 防火牆擋住連線時，正確做法是開必要 port，不是整個關掉。
- 新主機能登入，不代表已經準備好上線。

### 一句話總結

新主機準備的重點是先建立可觀察、可維護、可追蹤的基線，而不是急著裝服務。

---

## CentOS 7 到 Rocky Linux 的差異速查

| 主題 | CentOS 7 常見做法 | Rocky Linux 9/10 建議 |
| --- | --- | --- |
| 套件管理 | `yum install` | `dnf install` |
| 服務管理 | `systemctl` | `systemctl` |
| 網路管理 | `ifconfig`、舊式 network scripts | `ip`、`nmcli`、NetworkManager |
| 防火牆 | `firewalld` / iptables | `firewalld`、nftables 底層 |
| Log | `/var/log/messages`、journal | `journalctl` 搭配 `/var/log` |
| Python | Python 2 常見 | Python 3 |
| 安全更新 | CentOS 7 已 EOL | 使用仍支援的 Rocky Linux 9/10 |

## 30 天入門練習路線

| 天數 | 目標 | 練習 |
| --- | --- | --- |
| 1-3 | 登入與基本命令 | `whoami`, `pwd`, `ls`, `cd`, `cat /etc/os-release` |
| 4-6 | 檔案與路徑 | 建立、複製、改名、刪除練習檔 |
| 7-9 | 看檔與 vim | 用 `less`, `grep`, `tail`, `vim` 編筆記 |
| 10-12 | 權限 | 練 `chmod`, `chown`, `id`, `groups` |
| 13-15 | 程序 | 練 `ps`, `top`, `kill`, `jobs` |
| 16-18 | 磁碟 | 練 `lsblk`, `df`, `du`，理解 mount |
| 19-21 | bash | 練變數、alias、redirect、pipe |
| 22-24 | script | 寫一個檢查磁碟或 log 的 script |
| 25-27 | 套件與服務 | 用 `dnf` 安裝 nginx，systemd 管理 |
| 28-30 | 綜合檢查 | 做一份新主機基線 checklist |

## 最後總結

鳥哥原教材的價值在於學習順序很扎實：先從使用者與指令列開始，再進檔案、權限、程序、檔案系統、shell、帳號、套件、服務與儲存。2026 年不建議再以 CentOS 7 當主要練習環境，但這條學習路線仍然有效。把環境換成 Rocky Linux 9/10，命令以 `dnf`、`systemctl`、`journalctl`、`nmcli` 這些現代工具為主，並且每一堂都補上「操作前檢查、預期結果、排查方向、做完驗證」，就能把舊教材轉成 junior 真正能照著學的 Linux 基礎訓練。
