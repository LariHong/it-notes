# Linux 基礎學習訓練教材：Rocky Linux 2026 更新整理

## 來源

- 鳥哥 Linux 基礎學習訓練教材 CentOS 7.x 目錄：https://linux.vbird.org/linux_basic_train/centos7/
- 鳥哥 Linux 基礎學習訓練教材 CentOS 8.x 目錄：https://linux.vbird.org/linux_basic_train/centos8/
- Red Hat CentOS Linux EOL 說明：https://www.redhat.com/en/blog/centos-linux-end-life-centos-stream-and-new-red-hat-enterprise-linux-landscape
- Rocky Linux Release and Version Guide：https://wiki.rockylinux.org/rocky/version/

## 來源提醒與 2026 版本選擇

鳥哥這份教材原本是 CentOS 7.x 授課目錄，頁面標示最近更新時間是 2017/06/29；CentOS 8.x 訓練教材則標示到 2020/06/24。以 2026 年來看，CentOS Linux 7 已在 2024/06/30 結束生命週期，Red Hat 也說明 CentOS Linux 已不再是仍有社群維護更新的發行版。Rocky Linux 官方頁面則列出 Rocky Linux 9 仍支援到 2032/05/31，Rocky Linux 10 支援到 2035/05/31。

因此這份筆記採用這個策略：

- 觀念主軸沿用鳥哥 15 堂課的學習順序。
- 實作環境建議使用 Rocky Linux 9 或 Rocky Linux 10；若公司環境是 RHEL 9/10、AlmaLinux 9/10，大多數基礎命令也可套用。
- CentOS 7 的 `yum`、舊網卡命名、舊服務設定方式會以歷史脈絡補充；練習時優先使用 `dnf`、`systemctl`、`nmcli`、`firewall-cmd` 等現代 RHEL 系工具。
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
| 第 9 堂 | regex 與 shell script | 會 grep/sed/awk 入門、寫 script | 批次維運 |
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

### 實作練習

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

操作流程：

1. 登入 Rocky Linux VM。
2. 開啟終端機或透過 SSH 連線。
3. 執行上面的查詢命令。
4. 把輸出記到筆記：使用者名稱、主機名稱、發行版、kernel 版本。

### 負面例子 / 錯誤用法

錯誤做法：一開始就用 `root` 當日常帳號，所有操作都在最高權限下進行。

問題：

- 打錯刪除命令時可能直接毀掉系統。
- 看不出哪些命令真的需要權限。
- 未來接觸正式主機時容易養成危險習慣。

修正方向：平常使用一般帳號，需要管理權限時才用 `sudo`。

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

### 實作練習

```bash
# 範例用途：建立一個練習資料夾，複製檔案，再改名。
# 輸入說明：$HOME 是目前使用者家目錄，不需要硬背完整路徑。
# 輸出結果：會建立 linux-practice 目錄與 hello.txt / hello-backup.txt。
cd "$HOME"
mkdir -p linux-practice
cd linux-practice
printf "hello linux\n" > hello.txt
cp hello.txt hello-backup.txt
mv hello-backup.txt hello-2026.txt
ls -l
```

操作流程：

1. 先用 `pwd` 確認目前位置。
2. 在家目錄建立 `linux-practice`。
3. 用 `printf` 產生一個文字檔。
4. 複製檔案，再用 `mv` 改名。
5. 用 `ls -l` 檢查結果。

### 負面例子 / 錯誤用法

錯誤做法：

```bash
rm -rf *
```

問題：如果你剛好在錯的目錄，這會刪掉目前目錄下所有東西。

修正方向：

```bash
pwd
ls -la
rm -i hello-2026.txt
```

先確認位置，再用互動模式刪除重要檔案。

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

### 為什麼需要這個概念

Linux 錯誤訊息通常藏在 log，服務設定通常藏在 `/etc`。如果不會看檔案內容，就像工程師有監控卻不會讀儀表板。

### 核心重點

- `cat` 適合短檔案，`less` 適合長檔案。
- `tail -f` 適合即時追 log。
- `grep` 用關鍵字搜尋。
- vim 有一般模式、插入模式、命令模式；先會開檔、編輯、存檔、離開即可。

### 實作練習

```bash
# 範例用途：查系統版本檔、搜尋關鍵字，並用 vim 建立筆記。
# 輸入說明：/etc/os-release 是發行版資訊檔；VERSION 是搜尋關鍵字。
# 輸出結果：會顯示系統版本，並建立 notes.txt。
cat /etc/os-release
grep VERSION /etc/os-release
vim notes.txt
```

vim 最小操作：

1. `vim notes.txt` 開啟檔案。
2. 按 `i` 進入插入模式。
3. 輸入文字。
4. 按 `Esc` 回一般模式。
5. 輸入 `:wq` 存檔離開。
6. 如果改壞不想存，輸入 `:q!` 離開。

### 負面例子 / 錯誤用法

錯誤做法：直接編輯重要設定檔，沒有備份。

問題：改錯後服務可能啟不來，也不知道原本內容是什麼。

修正方向：

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak.$(date +%F)
sudo vim /etc/ssh/sshd_config
```

### Junior 常見誤解

- 卡在 vim 裡不是壞掉，通常只是還在一般模式或命令模式。
- `tail -f` 會持續追蹤檔案，要按 `Ctrl+C` 停止。
- `/etc` 多是系統設定，修改前要備份。

### 一句話總結

會找、會看、會改文字檔，是 Linux 排錯與維運的核心能力。

---

## 第 4 堂：Linux 基礎檔案權限與基礎帳號管理

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit04.php

### 這堂課主要在講什麼

Linux 權限的核心是「誰可以對什麼檔案做什麼事」。最基本的三種身份是 owner、group、others；三種動作是 read、write、execute。

### 為什麼需要這個概念

權限設太鬆會造成資料外洩或服務被竄改；設太緊又會讓程式無法讀設定、寫 log、執行檔案。你需要看懂 `ls -l` 的權限欄。

### 核心重點

- `r` 是讀取，`w` 是寫入，`x` 是執行或進入目錄。
- 檔案的 `x` 代表可執行；目錄的 `x` 代表可進入。
- `chmod` 改權限，`chown` 改擁有者，`chgrp` 改群組。
- 不要把 `chmod 777` 當萬用解法。

### 實作練習

```bash
# 範例用途：建立腳本，調整成只有擁有者可改、所有人可讀與執行。
# 輸入說明：755 代表 owner=rwx, group=rx, others=rx。
# 輸出結果：hello.sh 可以被執行，但只有 owner 能修改。
printf '#!/usr/bin/env bash\necho "hello permission"\n' > hello.sh
ls -l hello.sh
chmod 755 hello.sh
./hello.sh
```

### 負面例子 / 錯誤用法

錯誤做法：

```bash
sudo chmod -R 777 /var/www
```

問題：任何本機使用者或被入侵的服務帳號都可能改你的網站檔案。

修正方向：先確認服務使用者，再只給必要目錄寫入權限，例如只讓 upload/cache 目錄可寫。

### Junior 常見誤解

- `777` 不是「修好權限」，只是把門全打開。
- 目錄沒有 `x` 時，即使有 `r` 也不一定能進去查看內容。
- 使用者屬於哪些群組會影響實際權限，可用 `id` 查看。

### 一句話總結

Linux 權限不是背數字，而是判斷「哪個身份需要哪個最小動作」。

---

## 第 5 堂：權限應用、程序之觀察與基本管理

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit05.php

### 這堂課主要在講什麼

程序 process 是正在執行的程式。你要能看目前哪些程式在跑、誰啟動它、消耗多少 CPU/RAM，以及必要時如何停止。

### 為什麼需要這個概念

正式主機常見問題是 CPU 滿載、記憶體爆掉、服務卡住。你如果只會重開機，會看不到真正原因。

### 核心重點

- `ps aux` 看程序快照。
- `top` 或 `htop` 看即時資源。
- `kill` 送 signal，不是永遠等於強制刪除。
- SUID/SGID/SBIT 是特殊權限，能改變執行身份或目錄刪除規則。

### 實作練習

```bash
# 範例用途：建立一個背景 sleep 程序，查到它，再結束它。
# 輸入說明：sleep 300 會等待 300 秒；& 讓它在背景執行。
# 輸出結果：jobs 會看到背景工作，kill 會結束該程序。
sleep 300 &
jobs
ps -f
kill %1
jobs
```

### 負面例子 / 錯誤用法

錯誤做法：

```bash
sudo kill -9 1234
```

問題：`-9` 是強制終止，程式沒有機會收尾，可能造成暫存檔、lock、資料寫入中斷。

修正方向：先用一般 `kill PID`，再查 log；只有程序完全不回應時才考慮 `kill -9`。

### Junior 常見誤解

- PID 會變，不能把某次看到的 PID 寫死在腳本裡。
- CPU 高不一定是壞事，要看是否符合當下工作。
- `root` 可以看到與管理更多程序，但也更容易誤殺重要服務。

### 一句話總結

程序管理的重點是先觀察再處理，不要把重開機當成唯一排錯工具。

---

## 第 6 堂：基礎檔案系統管理

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit06.php

### 這堂課主要在講什麼

檔案系統是磁碟空間被組織成可存檔案的方式。你要知道裝置、分割區、檔案系統、掛載點之間的關係。

### 為什麼需要這個概念

空間不足、磁碟掛不上、開機進 emergency mode，通常都和檔案系統或 `/etc/fstab` 有關。這是伺服器維運很常見的底層問題。

### 核心重點

- `lsblk` 看磁碟與分割區。
- `df -h` 看已掛載檔案系統使用量。
- `du -sh` 看目錄占用。
- `/etc/fstab` 控制開機自動掛載，寫錯可能導致開機失敗。

### 實作練習

```bash
# 範例用途：查看磁碟結構與目前空間使用量。
# 輸入說明：不修改磁碟，只讀取狀態。
# 輸出結果：會看到磁碟、掛載點、容量與使用率。
lsblk
df -h
du -sh "$HOME"/*
```

進階練習請只在測試 VM 的新增磁碟上做：

```bash
# 危險提醒：mkfs 會格式化目標裝置，請勿對系統碟操作。
# /dev/sdb1 只是範例，實際要用 lsblk 確認。
sudo mkfs.xfs /dev/sdb1
sudo mkdir -p /mnt/labdata
sudo mount /dev/sdb1 /mnt/labdata
df -h /mnt/labdata
```

### 負面例子 / 錯誤用法

錯誤做法：看到 `/dev/sda` 就直接 `mkfs.xfs /dev/sda`。

問題：`/dev/sda` 很可能是系統碟，格式化後系統資料會消失。

修正方向：先 `lsblk -f`、確認磁碟大小、掛載點、UUID，再操作未使用的新磁碟。

### Junior 常見誤解

- `/dev/sdb` 是整顆磁碟，`/dev/sdb1` 才是常見分割區。
- 掛載只是把檔案系統接到目錄樹，不等於複製資料。
- `df` 看檔案系統剩餘空間，`du` 看目錄占用。

### 一句話總結

磁碟管理要先看懂裝置、檔案系統與掛載點，不要急著格式化。

---

## 第 7 堂：認識 bash 基礎與系統救援

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit07.php

### 這堂課主要在講什麼

bash 不只是打指令的地方，它有變數、環境變數、alias、命令替換、歷史紀錄。系統救援則是當開機或設定壞掉時，進入救援模式修復。

### 為什麼需要這個概念

很多工具依賴環境變數，例如 `PATH` 決定 shell 去哪裡找命令。救援能力則讓你在主機不能正常開機時仍有修復方向。

### 核心重點

- `echo $PATH` 看命令搜尋路徑。
- `alias` 可建立短命令，但不要讓它掩蓋危險操作。
- `.bashrc` 是互動 shell 常見設定檔。
- 救援模式通常用於修 `/etc/fstab`、root 密碼、boot loader、檔案系統問題。

### 實作練習

```bash
# 範例用途：觀察 bash 環境與建立暫時 alias。
# 輸入說明：alias 只影響目前 shell；重開終端機通常會消失。
# 輸出結果：ll 會變成 ls -alF 的簡寫。
echo "$SHELL"
echo "$PATH"
alias ll='ls -alF'
ll
```

### 負面例子 / 錯誤用法

錯誤做法：

```bash
alias rm='rm -rf'
```

問題：把刪除變得更危險，且未來你在別台機器操作時會產生錯覺。

修正方向：可以設定 `alias rm='rm -i'` 作為初學保護，但更重要的是養成先確認路徑的習慣。

### Junior 常見誤解

- 修改 `.bashrc` 後要重新開 shell 或 `source ~/.bashrc` 才生效。
- `PATH` 壞掉時，不是命令不存在，而是 shell 找不到。
- 救援模式不是日常操作環境，是修復用的最低限度環境。

### 一句話總結

bash 是你操作 Linux 的工作台，理解環境變數與設定檔會讓你少走很多彎路。

---

## 第 8 堂：bash 指令連續下達與資料流重導向

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit08.php

### 這堂課主要在講什麼

Linux 命令可以像積木一樣串起來。pipe `|` 把前一個命令的輸出交給下一個命令；redirect `>`、`>>` 把輸出寫到檔案。

### 為什麼需要這個概念

真正的 Linux 效率來自組合工具，而不是單一巨大工具。查 log、篩資料、產報表、排查服務都會用到 pipe 和 redirect。

### 核心重點

- `>` 覆蓋檔案，`>>` 追加檔案。
- `2>` 重導錯誤輸出。
- `|` 串接 stdout 到下一個命令。
- `tee` 可同時顯示並寫入檔案。

### 實作練習

```bash
# 範例用途：列出 /etc 中包含 ssh 的檔名，存成報告。
# 輸入說明：grep -i 忽略大小寫；tee 同時顯示並寫檔。
# 輸出結果：ssh-files.txt 會保存查詢結果。
ls /etc | grep -i ssh | tee ssh-files.txt
wc -l ssh-files.txt
```

### 負面例子 / 錯誤用法

錯誤做法：

```bash
some-command > important.txt
```

問題：`>` 會覆蓋原檔案，重要資料可能直接消失。

修正方向：不確定時先用 `>>` 追加，或先備份檔案。

### Junior 常見誤解

- pipe 傳的是文字輸出，不是把整個程式「接在一起」。
- `>` 和 `>>` 差一個字元但後果差很多。
- 錯誤訊息預設走 stderr，不一定會被 `>` 收進檔案。

### 一句話總結

pipe 和 redirect 是 Linux 把小工具組合成工作流程的關鍵。

---

## 第 9 堂：正規表示法與 shell script 初探

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit09.php

### 這堂課主要在講什麼

正規表示法 regex 用來描述文字模式，shell script 則把多個命令寫成可重複執行的流程。這堂課是從手動操作走向自動化的入口。

### 為什麼需要這個概念

你不可能每天手動查 100 個 log。會 regex 與 script 後，你可以把「查詢、篩選、判斷、輸出」固定成工具。

### 核心重點

- `grep` 搜尋文字模式。
- `^` 表示行首，`$` 表示行尾，`.` 表示任一字元。
- script 第一行常見 `#!/usr/bin/env bash`。
- 變數引用要加雙引號，減少空白與特殊字元造成的錯誤。

### 實作練習

```bash
# 範例用途：建立一個檢查磁碟使用率的簡單 script。
# 輸入說明：THRESHOLD 是警戒百分比，不含 % 符號。
# 輸出結果：列出根目錄使用率，超過門檻時顯示 WARNING。
cat > check-disk.sh <<'SCRIPT'
#!/usr/bin/env bash
set -u

THRESHOLD=80
USAGE=$(df / | awk 'NR==2 { gsub("%", "", $5); print $5 }')

if [ "$USAGE" -ge "$THRESHOLD" ]; then
  echo "WARNING: / usage is ${USAGE}%"
else
  echo "OK: / usage is ${USAGE}%"
fi
SCRIPT

chmod 755 check-disk.sh
./check-disk.sh
```

### 負面例子 / 錯誤用法

錯誤做法：

```bash
rm -rf $TARGET/*
```

問題：如果 `TARGET` 是空字串，命令可能變成危險刪除。

修正方向：

```bash
if [ -n "${TARGET:-}" ] && [ -d "$TARGET" ]; then
  rm -rf "$TARGET"/*
fi
```

### Junior 常見誤解

- shell script 不是只把命令貼在一起；要處理輸入、錯誤與邊界條件。
- regex 的 `.` 不是小數點，而是任一字元；要匹配真正小數點常用 `\.`。
- 變數不加引號在遇到空白檔名時很容易壞掉。

### 一句話總結

regex 幫你找文字規律，shell script 幫你把規律操作變成可重複流程。

---

## 第 10 堂：使用者管理與 ACL 權限設定

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit10.php

### 這堂課主要在講什麼

這堂課從單一帳號進入多人共管。你會建立使用者、設定群組、授權 sudo，並用 ACL 處理傳統 owner/group/others 不夠細的情境。

### 為什麼需要這個概念

正式環境不能大家共用 root 密碼。每個人應該用自己的帳號登入，需要權限時透過 sudo，這樣才查得到誰做了什麼。

### 核心重點

- `useradd` 建立使用者，`passwd` 設密碼。
- `usermod -aG wheel user` 在 RHEL 系常用來給 sudo 權限。
- `sudo` 是授權執行，不是切換人格無痕操作。
- ACL 可用 `setfacl` / `getfacl` 做細部權限。

### 實作練習

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
touch shared.txt
setfacl -m u:dev01:rw shared.txt
getfacl shared.txt
```

### 負面例子 / 錯誤用法

錯誤做法：多人共用同一個 `admin` 帳號。

問題：事後無法追蹤誰改了設定，也無法安全地移除離職者權限。

修正方向：每個人一個帳號，透過群組與 sudo 規則授權。

### Junior 常見誤解

- `su` 和 `sudo` 不同；`su` 是切換使用者，`sudo` 是用授權執行命令。
- 把人加進 `wheel` 前要確認公司規範，不是所有人都該有 sudo。
- ACL 很方便，但過度使用會讓權限難以盤點。

### 一句話總結

多人管理主機時，帳號、群組、sudo、ACL 是可追蹤與最小權限的基礎。

---

## 第 11 堂：基礎設定、備份、檔案壓縮打包與工作排程

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit11.php

### 這堂課主要在講什麼

這堂課把系統時間、主機名稱、壓縮打包、備份與排程放在一起。這些不是華麗技巧，但是真正維運時每天都會遇到。

### 為什麼需要這個概念

時間錯會讓 log 對不起來、憑證驗證失敗、排程在錯誤時間跑。備份不會做，出事時就只能祈禱。排程不會設，例行工作只能靠人手。

### 核心重點

- `timedatectl` 管理時間與時區。
- `tar` 打包，`gzip` / `xz` 壓縮。
- `cron` 適合傳統週期任務，systemd timer 是現代替代方案之一。
- 備份要驗證能不能還原。

### 實作練習

```bash
# 範例用途：查看時間設定，備份練習目錄。
# 輸入說明：Asia/Taipei 是台灣常用時區。
# 輸出結果：產生 linux-practice-backup.tar.gz。
timedatectl
sudo timedatectl set-timezone Asia/Taipei
tar -czf linux-practice-backup.tar.gz "$HOME/linux-practice"
tar -tzf linux-practice-backup.tar.gz | head
```

cron 練習：

```bash
# 每天 02:30 執行備份腳本的 crontab 範例，不要直接照貼到正式環境。
30 2 * * * /home/student/bin/backup.sh >> /home/student/backup.log 2>&1
```

### 負面例子 / 錯誤用法

錯誤做法：只建立備份檔，從不測試解壓與還原。

問題：等真的出事才發現備份檔壞掉、路徑錯誤或根本沒有備到重要資料。

修正方向：定期在另一個目錄或測試機還原一次。

### Junior 常見誤解

- 壓縮不等於備份；備份還包含版本、保存位置、還原驗證。
- cron 的環境變數很少，script 裡最好使用絕對路徑。
- 系統時間不是小事，排錯時 log 時間線非常重要。

### 一句話總結

基礎設定與備份排程看似平凡，卻是系統能不能長期可靠運作的底盤。

---

## 第 12 堂：軟體管理與安裝及登錄檔初探

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit12.php

### 這堂課主要在講什麼

CentOS 7 教材會提 `rpm` 與 `yum`；2026 年在 Rocky/RHEL 系建議以 `dnf` 為主，`rpm` 用來查詢底層套件資訊。登錄檔則從傳統 `/var/log` 與 systemd journal 兩邊看。

### 為什麼需要這個概念

安裝軟體不是去網站下載 exe，而是透過套件庫取得可追蹤、可更新、可移除的 package。出錯時，log 是第一現場。

### 核心重點

- `dnf search` 搜尋套件。
- `dnf install` 安裝，`dnf remove` 移除，`dnf update` 更新。
- `rpm -q` 查套件是否安裝。
- `journalctl` 查 systemd journal。

### 實作練習

```bash
# 範例用途：安裝 tree，確認來源與版本，再移除。
# 輸入說明：tree 是顯示目錄樹的小工具。
# 輸出結果：tree 可顯示目錄結構；rpm 可查安裝狀態。
sudo dnf search tree
sudo dnf install -y tree
rpm -q tree
tree -L 2 "$HOME"
sudo dnf remove -y tree
```

查 log：

```bash
# 範例用途：查看最近一次開機以來的錯誤等級 log。
journalctl -p err -b
```

### 負面例子 / 錯誤用法

錯誤做法：隨便加入不明第三方 repository，再用 root 安裝套件。

問題：可能引入惡意或不相容套件，造成系統更新衝突。

修正方向：優先使用官方 repo；需要 EPEL 等第三方 repo 時，確認來源、用途與風險。

### Junior 常見誤解

- `dnf` 是高階套件管理工具，`rpm` 是底層套件工具；日常安裝優先用 `dnf`。
- 更新系統前要知道服務影響，正式環境不要無腦自動大更新。
- log 很多不代表都有問題，要看時間、服務、等級與上下文。

### 一句話總結

套件管理讓軟體可追蹤，log 則讓問題有線索可查。

---

## 第 13 堂：服務管理與開機流程管理

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit13.php

### 這堂課主要在講什麼

現代 RHEL 系使用 systemd 管理服務與開機流程。你要會啟動、停止、重啟、設定開機自動啟動、查服務狀態與 log。

### 為什麼需要這個概念

伺服器不是只把程式跑起來就好，還要確保重開機後服務會回來、失敗時看得到原因、設定修改後能安全 reload。

### 核心重點

- `systemctl status service` 查狀態。
- `systemctl start/stop/restart/reload` 控制服務。
- `systemctl enable --now` 設定立即啟動且開機自動啟動。
- `journalctl -u service` 查某服務 log。

### 實作練習

```bash
# 範例用途：安裝並啟動 nginx，查看服務狀態與 log。
# 輸入說明：nginx 是常見 web server；需要 sudo 權限。
# 輸出結果：nginx 服務啟動，狀態顯示 active。
sudo dnf install -y nginx
sudo systemctl enable --now nginx
systemctl status nginx
journalctl -u nginx -b --no-pager
```

### 負面例子 / 錯誤用法

錯誤做法：改完設定後直接 `restart`，沒有先檢查設定。

問題：設定檔錯誤會讓服務停掉後起不來。

修正方向：能檢查設定的服務先跑測試，例如 nginx：

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### Junior 常見誤解

- `start` 只啟動當下，不代表下次開機會自動啟動。
- `enable` 只設定開機自動，不一定立刻啟動；可用 `enable --now`。
- `status` 顯示的最後幾行 log 不一定夠，要搭配 `journalctl`。

### 一句話總結

systemd 是現代 Linux 服務管理入口，會用它才能讓服務可啟動、可追蹤、可維護。

---

## 第 14 堂：進階檔案系統管理

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit14.php

### 這堂課主要在講什麼

這堂課處理 RAID、LVM、Quota 等進階儲存主題。初學者不用急著全部實作到熟，但要知道它們分別解決什麼問題。

### 為什麼需要這個概念

伺服器資料會成長，磁碟會壞，多人共用空間會有人用爆。RAID 處理可用性與效能，LVM 處理彈性配置，Quota 處理使用量限制。

### 核心重點

- Software RAID 用 `mdadm` 管理多顆磁碟組合。
- LVM 把 PV、VG、LV 分層，讓容量管理更彈性。
- Quota 可限制使用者或群組用量。
- RAID 不是備份，LVM snapshot 也不是長期備份。

### 實作練習

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

1. PV：把實體磁碟或分割區交給 LVM。
2. VG：把多個 PV 合併成容量池。
3. LV：從容量池切出可格式化與掛載的邏輯磁碟。
4. Filesystem：在 LV 上建立 XFS/ext4。
5. Mount point：掛載到 `/data` 這類目錄使用。

### 負面例子 / 錯誤用法

錯誤做法：以為 RAID1 就等於備份。

問題：RAID1 只能在單顆磁碟壞掉時維持可用；如果你誤刪資料、被勒索軟體加密、檔案系統損毀，錯誤也會同步到鏡像。

修正方向：RAID 管可用性，備份管可還原性；兩者都要規劃。

### Junior 常見誤解

- LVM 讓擴充容量方便，但不是叫你不做容量規劃。
- Quota 是限制用量，不是解決磁碟空間不足的唯一方法。
- 儲存操作風險高，正式環境要先備份與演練。

### 一句話總結

進階儲存管理是在容量、可用性與使用限制之間做工程取捨。

---

## 第 15 堂：Linux 系統的準備 Optional

原文連結：https://linux.vbird.org/linux_basic_train/centos7/unit15.php

### 這堂課主要在講什麼

這堂課像是把一台新 Linux 主機整理成可用狀態：確認用途、安裝系統、建立帳號、更新套件、設定網路、SSH、防火牆、時間、服務與基本安全。

### 為什麼需要這個概念

很多事故不是因為高深技術，而是新主機一開始就沒有基線：沒更新、共用 root、SSH 密碼登入、時區錯、防火牆亂開、沒有備份。

### 核心重點

- 先定義主機用途，再裝套件與開服務。
- 初始設定包含更新、帳號、SSH、時間、防火牆、SELinux、log、備份。
- 不要一開始就關閉安全機制。
- 建議把基線設定寫成 checklist 或自動化腳本。

### 實作練習：新主機基線 checklist

```bash
# 範例用途：新 Rocky Linux 主機安裝後的基礎檢查。
# 輸入說明：這些命令多數只讀狀態；更新與設定需要 sudo。
# 輸出結果：確認版本、更新、時間、防火牆、SELinux、SSH 服務狀態。
cat /etc/os-release
sudo dnf update -y
timedatectl
systemctl status firewalld
getenforce
systemctl status sshd
```

建議流程：

1. 建立一般管理帳號，確認可 sudo。
2. 更新系統套件。
3. 設定時區與 NTP。
4. 確認 SSH 登入策略。
5. 啟用 firewalld，只開必要 port。
6. 保持 SELinux enforcing，遇到問題先查 audit log。
7. 設定備份與監控。
8. 記錄主機用途、IP、開放服務、管理者。

### 負面例子 / 錯誤用法

錯誤做法：

```bash
sudo setenforce 0
sudo systemctl stop firewalld
```

問題：這只是把安全檢查拿掉，不是解決問題。正式環境會增加入侵風險，也讓未來問題更難追。

修正方向：查明是 SELinux policy、防火牆 port、服務 bind address 還是權限問題，再做最小範圍調整。

### Junior 常見誤解

- 主機能 ping 不代表服務可用；要確認 port、服務狀態、防火牆與應用 log。
- 關防火牆只是測試手段，不是長期設定。
- 安裝完成不是結束，而是維護生命週期的開始。

### 一句話總結

一台好維護的 Linux 主機，從安裝後第一小時的基線設定就開始決定命運。

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
| 安全狀態 | CentOS 7 已 EOL | Rocky 9/10 仍在支援週期 |

## 30 天入門練習路線

| 天數 | 目標 | 練習 |
| --- | --- | --- |
| 1-3 | 登入與基本命令 | `whoami`, `pwd`, `ls`, `cd`, `cat /etc/os-release` |
| 4-6 | 檔案管理 | 建立、複製、搬移、刪除測試檔 |
| 7-9 | 看檔與 vim | 用 `less`, `grep`, `tail`, `vim` 編筆記 |
| 10-12 | 權限 | 練 `chmod`, `chown`, `id`, `groups` |
| 13-15 | 程序 | 練 `ps`, `top`, `kill`, `jobs` |
| 16-18 | 磁碟 | 練 `lsblk`, `df`, `du`，理解 mount |
| 19-21 | pipe/script | 寫 2 個小 shell script |
| 22-24 | 使用者與 sudo | 建帳號、群組、ACL |
| 25-27 | 套件與服務 | 用 `dnf` 安裝 nginx，systemd 管理 |
| 28-30 | 綜合演練 | 新 VM 基線設定、備份、查 log |

## 最後總結

鳥哥原教材的價值在於學習順序很扎實：先從使用者與指令列開始，再進檔案、權限、程序、檔案系統、shell、帳號、套件、服務與儲存。2026 年不建議再以 CentOS 7 當主要練習環境，但這條學習路線仍然有效。把環境換成 Rocky Linux 9/10，命令以 `dnf`、`systemctl`、`journalctl`、`nmcli` 這些現代工具為主，就能把舊教材轉成仍適合現在工作的 Linux 基礎訓練。
