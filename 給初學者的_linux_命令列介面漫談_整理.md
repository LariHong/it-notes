# 給初學者的 Linux 命令列介面漫談：教學整理筆記

來源系列：[給初學者的 linux 命令列介面漫談](https://ithelp.ithome.com.tw/users/20152654/ironman/7703)，作者：tjwei，2024 iThome 鐵人賽，影片教學組，共 31 篇。

來源 RSS：[https://ithelp.ithome.com.tw/rss/series/7703](https://ithelp.ithome.com.tw/rss/series/7703)

## 這個系列在教什麼

這個系列不是只教幾個 Linux 指令，而是在建立一種工作方式：遇到檔案、資料、網路、系統狀態、遠端機器、AI 工具或終端機 UI 時，先思考「能不能用命令列快速組合工具完成」。

對初學者來說，命令列最卡的地方通常不是指令本身，而是這三件事：

- 不知道每個工具適合解決什麼問題。
- 不知道指令之間如何用 pipe、檔案、JSON、CSV 串接。
- 害怕指令會破壞系統，所以只敢複製貼上。

這份筆記會把 31 篇整理成可練習的路線。每一篇都附上「可以自己打的指令」、「適合情境」、「不適合情境」、「踩雷點」，讓你讀到哪一篇就能開始動手。

## 系列閱讀路線

| 階段 | 對應 Day | 目標 |
| --- | --- | --- |
| 入門環境與終端機概念 | Day 1-2 | 了解終端機、VS Code terminal、sixel 圖片顯示 |
| 傳統 Unix 工具 | Day 3-6 | 學會文字處理、job control、process、磁碟與網路檢查 |
| 生產力工具 | Day 7-13 | tmux、nix、vim、Python、jq、visidata、下載工具、瀏覽器、Windows Terminal |
| 開發與遠端工作 | Day 14-18 | Docker、SSH、ffmpeg、Python `-m`、終端機看 YouTube |
| 現代 CLI 生態 | Day 19-25 | CLI 文藝復興、Nushell、檔案管理、天氣 API、xonsh、Elvish |
| 進階資料與展示 | Day 26-31 | bash history、DuckDB、資源清單、x-cmd、終端機簡報、中文 TTE |

## 先建立直覺：命令列像樂高

GUI 工具像是一台完整機器，按鈕都幫你排好了；命令列工具比較像樂高，每個工具只做一件事，但你可以把它們組成自己的流程。

例如：

```bash
curl -s "https://api.example.com/data.json" \
  | jq '.items[] | select(.score > 80) | {name, score}' \
  | tee result.json
```

這段流程同時做了三件事：

1. `curl` 抓資料。
2. `jq` 篩選與轉換 JSON。
3. `tee` 一邊顯示結果，一邊存檔。

這就是命令列的核心價值：把小工具組成可重複、可記錄、可自動化的工作流程。

## Day 1：為什麼選 VS Code terminal，而不是 Cursor 或 Warp

原文：[命令列漫談 1](https://ithelp.ithome.com.tw/articles/10349756?sc=rss.iron)

### 核心概念

第一篇先把命令列放回開發情境：終端機不是獨立存在的黑盒子，而是你和 Git、Docker、雲端、Linux 系統互動的入口。作者也提到 `cd`、`pushd`、`popd`、`ln -s`，並說明為什麼使用 VS Code 作為 terminal emulator。

### 實作練習

```bash
pwd
cd /tmp
mkdir cli-demo
cd cli-demo
pushd /var/log
popd
ln -s /tmp/cli-demo demo-link
ls -l demo-link
```

### 適合使用情境

- 你正在學 Git、Docker、Linux 或雲端工具。
- 你希望在編輯器裡一邊寫程式、一邊跑指令。
- 你想把常用目錄用 symbolic link 做成更短的入口。

### 不適合使用情境與踩雷點

- 不要把 `ln -s` 當成複製檔案。symbolic link 是指向原位置，原檔移走就會失效。
- 不要在不確定目前目錄時執行刪除或搬移指令。先 `pwd`、`ls` 確認。

### Junior 常見誤解

「我會用 VS Code，所以不用學命令列」是很常見的誤會。VS Code 可以包住終端機，但底層工具仍然是命令列；越早理解，越不會被工具介面限制。

## Day 2：sixel，在文字終端機顯示圖片

原文：[命令列漫談 2](https://ithelp.ithome.com.tw/articles/10350068?sc=rss.iron)

### 核心概念

sixel 是一種讓終端機顯示點陣圖片的技術。傳統終端機主要顯示文字，但現代終端機開始支援圖片、圖表、預覽與更豐富的互動體驗。

### 實作練習

```bash
# 需要安裝 ImageMagick 與支援 sixel 的終端機
magick input.png -resize 80x sixel:-

# 若有 lsix
lsix *.png
```

### 適合使用情境

- 在遠端機器快速預覽圖片，不想下載到本機。
- 資料科學、影像處理、報表檢查時想在 terminal 直接看結果。

### 不適合使用情境與踩雷點

- 不要假設所有終端機都支援 sixel。macOS Terminal、某些 SSH client 或舊版 terminal 可能不能顯示。
- 不要把 terminal 圖片預覽當成正式 UI。它適合快速檢查，不適合精準排版。

### Junior 常見誤解

終端機不是只能顯示純文字。它的能力取決於 terminal emulator、字型、協定支援與工具鏈。

## Day 3：傳統的命令列資料處理工具

原文：[命令列漫談 3](https://ithelp.ithome.com.tw/articles/10350074?sc=rss.iron)

### 核心概念

`head`、`tail`、`sort`、`uniq`、`wc`、`cut`、`diff`、`shuf`、`grep`、`sed`、`tr` 是 Unix 傳統資料處理工具。它們的共同特色是：輸入文字，輸出文字，適合串接。

### 實作練習

```bash
head data.csv
tail data.csv
cut -d, -f1 data.csv | sort | uniq -c | sort -nr
grep "ERROR" app.log | wc -l
diff old.txt new.txt
sed 's/foo/bar/g' input.txt
tr '[:lower:]' '[:upper:]' < names.txt
```

### 適合使用情境

- 快速檢查大型文字檔。
- 從 log、CSV、設定檔中找線索。
- 寫一次性資料清理流程。

### 不適合使用情境與踩雷點

- `cut -d,` 不是真正的 CSV parser，遇到引號內含逗號會出錯。
- `sed` 大量替換前先輸出到新檔，不要直接覆蓋原始資料。

### Junior 常見誤解

傳統工具不代表過時。它們簡單、快、容易組合；真正要注意的是資料格式是否適合用純文字方式處理。

## Day 4：命令列的 job control

原文：[命令列漫談 4](https://ithelp.ithome.com.tw/articles/10350889?sc=rss.iron)

### 核心概念

job control 是在同一個 shell 裡管理前景、背景與暫停中的工作。常見工具包含 `&`、`jobs`、`fg`、`bg`、`Ctrl+Z`、`disown`、`nohup`。

### 實作練習

```bash
sleep 100 &
jobs
fg %1
# 按 Ctrl+Z 暫停
bg %1
nohup bash -c 'while true; do date; sleep 5; done' > timer.log 2>&1 &
disown
```

### 適合使用情境

- 遠端機器上跑長時間任務。
- 需要暫停某個互動程式，先切出去做別的事。

### 不適合使用情境與踩雷點

- 重要長任務只用 `&` 不夠，登出後可能被終止。
- 長期服務應該用 systemd、supervisor、container 或排程系統管理，不要只靠 shell job。

### Junior 常見誤解

背景執行不等於「永遠執行」。它仍可能受到 terminal session、signal、stdin/stdout 的影響。

## Day 5：系統資訊上篇，Process

原文：[命令列漫談 5](https://ithelp.ithome.com.tw/articles/10351025?sc=rss.iron)

### 核心概念

process 是正在執行的程式實例。`w`、`who`、`uptime`、`top`、`htop`、`ps`、`pstree`、`kill`、`killall` 可以幫你觀察與管理系統狀態。

### 實作練習

```bash
uptime
who
ps aux | head
pstree -p
top
kill -TERM <pid>
kill -KILL <pid>
```

### 適合使用情境

- 查 CPU 或記憶體被誰吃掉。
- 找卡住的程式。
- 理解 parent process 與 child process。

### 不適合使用情境與踩雷點

- 不要一開始就 `kill -9`。先用 `TERM`，讓程式有機會清理資源。
- 不要看到高 CPU 就直接殺 process。先確認是不是正常工作負載。

### Junior 常見誤解

`kill` 不是只有「殺掉」的意思，它本質上是送 signal。`TERM`、`INT`、`KILL` 的語意不同。

## Day 6：系統資訊下篇，儲存空間與網路

原文：[命令列漫談 6](https://ithelp.ithome.com.tw/articles/10352698?sc=rss.iron)

### 核心概念

這篇把系統觀察擴展到磁碟、硬體、記憶體與網路。常見工具有 `df`、`du`、`ncdu`、`uname`、`free`、`lshw`、`lsusb`、`lspci`、`ip`、`ss`、`fuser`、`ping`、`tracepath`、`nc`、`nmap`。

### 實作練習

```bash
df -h
du -sh .
free -h
ip addr
ss -tulpn
ping -c 4 8.8.8.8
tracepath example.com
```

### 適合使用情境

- 磁碟滿了，要找是哪個目錄變大。
- 服務開不起來，要查 port 是否被占用。
- 網路連不上，要分辨 DNS、路由或服務問題。

### 不適合使用情境與踩雷點

- `nmap` 不要亂掃不屬於自己的網段或主機。
- `du` 掃大目錄可能很慢，正式機器上要注意負載。

### Junior 常見誤解

「網路不通」不是一種問題，而是一串可能問題：本機 IP、DNS、路由、防火牆、遠端服務、TLS 都可能是原因。

## Day 7：tmux、nix、vim

原文：[命令列漫談 7](https://ithelp.ithome.com.tw/articles/10353480?sc=rss.iron)

### 核心概念

這篇介紹三個強但不必一次學完的工具：`tmux` 管理終端機 session，`nix-shell` 建立臨時工具環境，`vim` 是終端機編輯器。

### 實作練習

```bash
tmux new -s work
# tmux 中按 Ctrl+b 再按 % 分割窗格
# Ctrl+b 再按 d detach
tmux attach -t work

nix-shell -p jq ripgrep
jq --version
rg --version

vim notes.txt
```

### 適合使用情境

- SSH 到遠端機器工作，避免網路斷線就失去 session。
- 暫時試用工具，不想污染系統環境。
- 在沒有 GUI 的環境快速編輯檔案。

### 不適合使用情境與踩雷點

- 不要一開始就追求完整 vim 配置。先學開檔、編輯、儲存、離開。
- `nix` 很強，但學習曲線不低；初學先從 `nix-shell -p` 開始。

### Junior 常見誤解

工具不用一次精通。先學能救急的 20%，等遇到真問題再補進階功能。

## Day 8：命令列下的 Python

原文：[命令列漫談 8](https://ithelp.ithome.com.tw/articles/10354370?sc=rss.iron)

### 核心概念

Python 不只可以寫 `.py` 檔，也可以是終端機裡的互動工具。文章提到 Python 原生 REPL、bpython、IPython、Rich、Euporie、Jupyter Console。

### 實作練習

```bash
python
python - <<'PY'
from pathlib import Path
print(sum(1 for _ in Path(".").glob("*")))
PY

ipython
jupyter console
```

### 適合使用情境

- 快速驗證語法或資料處理想法。
- 需要比 shell 更穩定的字串、JSON、日期或數學處理。
- 終端機中做輕量分析或圖表檢查。

### 不適合使用情境與踩雷點

- 不要把長而複雜的 one-liner 塞在 shell 裡。可讀性差時就寫成 `.py`。
- 在 production script 裡要固定 Python 版本與相依套件。

### Junior 常見誤解

shell 和 Python 不是互斥。shell 適合串工具，Python 適合處理複雜邏輯。

## Day 9：jq，快速處理 JSON 的工具

原文：[命令列漫談 9](https://ithelp.ithome.com.tw/articles/10355019?sc=rss.iron)

### 核心概念

`jq` 是 JSON 的查詢、篩選與轉換工具。遇到 API、雲端工具、LLM 回應、標註資料時，JSON 幾乎無所不在。

### 實作練習

```bash
cat data.json | jq .
cat data.json | jq '.records | keys'
cat data.json | jq '.items[] | {name, value}'
cat data.json | jq '.items[] | select(.temperature > 33)'
```

### 適合使用情境

- 快速理解 API 回傳結構。
- 從 JSON 抽出欄位。
- 在 shell pipeline 中做輕量資料分析。

### 不適合使用情境與踩雷點

- 大型資料或複雜邏輯可能改用 Python、DuckDB 或資料庫比較好。
- 不要把不可信 JSON 內容直接拼進 shell 指令執行。

### Junior 常見誤解

`jq` 不只是 pretty print。真正威力在 filter、map、select、group、reduce。

## Day 10：終端機 DataFrame 檢視器 visidata

原文：[命令列漫談 10](https://ithelp.ithome.com.tw/articles/10355544?sc=rss.iron)

### 核心概念

visidata 是終端機中的資料檢視與處理工具，可處理 CSV、Parquet、網頁表格等資料，介於 spreadsheet、dataframe viewer 和 shell 工具之間。

### 實作練習

```bash
vd data.csv
vd data.parquet
vd https://en.wikipedia.org/wiki/List_of_countries_by_population_(United_Nations)
```

### 適合使用情境

- 快速看資料長什麼樣子。
- 在 SSH 環境檢查 CSV 或 Parquet。
- 不想先開 Jupyter 或 Excel。

### 不適合使用情境與踩雷點

- 正式資料轉換流程仍應寫成可重跑的 script。
- 互動式操作很方便，但要記得記錄你做過的清理步驟。

### Junior 常見誤解

能互動操作不代表不可工程化。用 visidata 探索資料後，仍應把正式流程整理成 script 或 notebook。

## Day 11：資料下載工具

原文：[命令列漫談 11](https://ithelp.ithome.com.tw/articles/10356281?sc=rss.iron)

### 核心概念

下載不是只有瀏覽器。`curl`、`wget`、`gdown`、`rclone`、`rsync`、`scp` 分別適合 HTTP、雲端硬碟、同步、遠端複製等任務。

### 實作練習

```bash
curl -L -o file.zip "https://example.com/file.zip"
wget -O file.zip "https://example.com/file.zip"
rsync -av source/ user@host:/data/source/
scp report.txt user@host:/tmp/
rclone copy remote:dataset ./dataset
```

### 適合使用情境

- 自動下載資料集。
- 遠端同步目錄。
- 需要續傳、同步、掛載雲端儲存。

### 不適合使用情境與踩雷點

- `rsync source/ dest/` 和 `rsync source dest/` 意義不同，斜線很重要。
- 下載執行檔後不要直接執行，先確認來源與 checksum。

### Junior 常見誤解

下載工具不只是「抓檔案」。很多時候它們處理的是驗證、續傳、同步、權限與遠端協定。

## Day 12：在文字終端機瀏覽網頁

原文：[命令列漫談 12](https://ithelp.ithome.com.tw/articles/10356804?sc=rss.iron)

### 核心概念

文字瀏覽器讓你在 terminal 中閱讀網頁、測試內容、在 SSH 環境查文件。常見工具包括 `lynx`、`w3m`、`elinks`、`browsh` 等。

### 實作練習

```bash
lynx https://example.com
w3m https://example.com
curl -L https://example.com | sed -n '1,40p'
```

### 適合使用情境

- 遠端伺服器沒有 GUI，但需要查文件。
- 想快速看 HTML 文字內容。
- 測試網站在無 JavaScript、低功能環境下的可讀性。

### 不適合使用情境與踩雷點

- 現代重 JavaScript 網站可能無法正常使用。
- 涉及登入、金融、敏感資訊時要注意 cookie 與終端機紀錄。

### Junior 常見誤解

文字瀏覽器不是取代 Chrome，而是提供另一種低依賴、可腳本化的閱讀方式。

## Day 13：Windows Terminal 支援 sixel 了

原文：[終端機漫談 13](https://ithelp.ithome.com.tw/articles/10357580?sc=rss.iron)

### 核心概念

Windows Terminal 對 sixel 的支援代表 Windows 命令列體驗越來越接近 Unix-like 開發環境。終端機能力不只取決於 shell，也取決於 terminal emulator。

### 實作練習

```powershell
wsl --install
wsl
sudo apt update
sudo apt install imagemagick
magick input.png -resize 80x sixel:-
```

### 適合使用情境

- Windows 使用者想用 WSL 做 Linux 開發。
- 需要在 Windows Terminal 中預覽圖片或資料圖。

### 不適合使用情境與踩雷點

- PowerShell、CMD、WSL shell 是不同層次，不要混淆。
- 指令在 Windows 原生與 WSL 中路徑格式不同。

### Junior 常見誤解

「Windows 不能學 Linux 命令列」已經不太成立。WSL 加 Windows Terminal 是很實用的入門組合。

## Day 14：Docker 的基本指令

原文：[命令列漫談 14](https://ithelp.ithome.com.tw/articles/10358298?sc=rss.iron)

### 核心概念

Docker 讓你用 container 包住執行環境。初學者至少要懂 image、container、volume、port mapping 與 log。

### 實作練習

```bash
docker pull nginx
docker run --name web -p 8080:80 -d nginx
docker ps
docker logs web
docker exec -it web sh
docker stop web
docker rm web
```

### 適合使用情境

- 快速啟動資料庫、Web server 或測試服務。
- 團隊需要一致的開發環境。

### 不適合使用情境與踩雷點

- 不要把 container 當 VM。container 通常應該是可重建、可替換的。
- 資料庫資料要掛 volume，否則刪 container 可能一起消失。

### Junior 常見誤解

`docker run` 成功只是第一步。正式使用還要理解網路、volume、環境變數、image tag 與安全更新。

## Day 15：用 SSH 取代 VPN

原文：[命令列漫談 15](https://ithelp.ithome.com.tw/articles/10358883?sc=rss.iron)

### 核心概念

SSH 不只登入遠端主機，也能做 port forwarding、proxy、跳板機與遠端服務存取。

### 實作練習

```bash
ssh user@server
ssh -L 8080:localhost:80 user@server
ssh -N -D 1080 user@server
ssh -J jumpuser@jump-host user@private-host
```

### 適合使用情境

- 臨時存取遠端內網服務。
- 用跳板機進入受保護環境。
- 開發或除錯時建立安全 tunnel。

### 不適合使用情境與踩雷點

- SSH tunnel 不等於完整 VPN，權限、路由與審計能力不同。
- 不要共用 private key，不要把 key 放進 repo。

### Junior 常見誤解

SSH 是安全通道，但安全性仍取決於 key 管理、使用者權限、伺服器設定與網路政策。

## Day 16：用 ffmpeg、ImageMagick 編輯影像

原文：[命令列漫談 16](https://ithelp.ithome.com.tw/articles/10359469?sc=rss.iron)

### 核心概念

`ffmpeg` 處理影音，ImageMagick 處理圖片。它們適合批次轉檔、縮圖、裁切、抽影格、加浮水印。

### 實作練習

```bash
ffmpeg -i input.mp4 -vf "scale=1280:-1" output.mp4
ffmpeg -i input.mp4 -r 1 frames/frame_%03d.png
magick input.png -resize 512x512 output.png
magick *.png output.pdf
```

### 適合使用情境

- 大量圖片縮放。
- 從影片抽 frame 做資料集。
- 自動產生縮圖或轉格式。

### 不適合使用情境與踩雷點

- 不熟參數時不要直接覆蓋原檔。
- 影片編碼會影響畫質、大小與相容性，不能只看副檔名。

### Junior 常見誤解

影音處理不是「轉檔就好」。容器格式、codec、bitrate、解析度、frame rate 都會影響結果。

## Day 17：Python `-m` 的各種隱藏命令列工具

原文：[命令列漫談 17](https://ithelp.ithome.com.tw/articles/10360101?sc=rss.iron)

### 核心概念

`python -m module` 可以直接把模組當成命令列工具執行。這是 Python 標準庫與套件提供 CLI 的常見方式。

### 實作練習

```bash
python -m http.server 8000
python -m json.tool data.json
python -m venv .venv
python -m pip list
python -m timeit "sum(range(1000))"
```

### 適合使用情境

- 臨時開靜態檔案伺服器。
- 格式化 JSON。
- 建立虛擬環境。
- 快速量測小段 Python。

### 不適合使用情境與踩雷點

- `python -m http.server` 不適合公開到網際網路。
- 多版本 Python 環境要確認 `python` 指到哪個版本。

### Junior 常見誤解

Python 不是只有寫程式檔。標準庫本身就藏了很多可直接用的 CLI。

## Day 18：在終端機看 YouTube

原文：[命令列漫談 18](https://ithelp.ithome.com.tw/articles/10360585?sc=rss.iron)

### 核心概念

這篇展示終端機與影音工具的組合，例如用 `mpv` 播放、用 `yt-dlp` 取得影片資訊、用 `timg` 或其他工具顯示影像。

### 實作練習

```bash
yt-dlp --get-title "https://www.youtube.com/watch?v=VIDEO_ID"
yt-dlp -f bestaudio "https://www.youtube.com/watch?v=VIDEO_ID"
mpv "https://www.youtube.com/watch?v=VIDEO_ID"
```

### 適合使用情境

- 在低資源環境播放影音。
- 把影片資訊整合進腳本。
- 下載授權允許的教學影片或素材。

### 不適合使用情境與踩雷點

- 要尊重平台條款與版權。
- 串接第三方影片平台時，指令可能因平台改版而失效。

### Junior 常見誤解

命令列工具能做很多事，但不是所有「做得到」都等於「應該做」。授權、條款和來源可信度都要看。

## Day 19：命令列文藝復興

原文：[命令列漫談 19](https://ithelp.ithome.com.tw/articles/10361217?sc=rss.iron)

### 核心概念

現代 CLI 工具大量出現，常見特色是更好的 UI、更好的錯誤訊息、JSON/YAML 支援、跨平台、和 AI 或雲端工具整合。這不是取代傳統工具，而是擴充命令列的可能性。

### 實作練習

```bash
bat README.md
fd "test"
rg "TODO"
eza -la
zoxide query project
```

### 適合使用情境

- 想提升日常查檔、看檔、搜尋效率。
- 希望工具輸出更適合人閱讀。

### 不適合使用情境與踩雷點

- script 中不要過度依賴漂亮輸出，優先使用穩定格式。
- 團隊環境要確認每個人都安裝同一批工具。

### Junior 常見誤解

新工具不是因為舊工具沒用，而是針對現代需求做了更好的預設值。

## Day 20：新風格的殼 Nushell

原文：[命令列漫談 20](https://ithelp.ithome.com.tw/articles/10361760?sc=rss.iron)

### 核心概念

Nushell 把資料視為結構化表格，而不是單純文字。這讓 pipeline 可以操作欄位、列、型別，接近 dataframe 的思維。

### 實作練習

```nu
ls | where size > 10mb
open data.csv | where score > 80 | sort-by score
sys | get host
```

### 適合使用情境

- 常處理表格、JSON、CSV。
- 想減少文字 parsing 的脆弱性。

### 不適合使用情境與踩雷點

- 傳統 shell script 不能直接搬到 Nushell。
- 團隊或伺服器上未必有 Nushell，部署前要確認環境。

### Junior 常見誤解

Nushell 不是 Bash 的語法糖，而是不同資料模型的 shell。

## Day 21：檔案管理工具 zoxide、yazi、mc

原文：[命令列漫談 21](https://ithelp.ithome.com.tw/articles/10362338?sc=rss.iron)

### 核心概念

`zoxide` 幫你快速跳目錄，`yazi` 是現代 terminal file manager，`mc` 是經典雙欄檔案管理器。它們解決的是「在 terminal 裡移動與管理檔案」的效率問題。

### 實作練習

```bash
zoxide add ~/projects/myapp
z myapp
yazi
mc
```

### 適合使用情境

- 專案目錄很多，常常切換。
- SSH 環境中需要檔案管理器。
- 不想每次都手打長路徑。

### 不適合使用情境與踩雷點

- 批次搬移或大量刪除仍建議先 dry run 或用 script 記錄。
- 互動式工具很方便，但操作紀錄不如 script 清楚。

### Junior 常見誤解

命令列不是只能手打路徑。成熟使用者會善用跳轉、模糊搜尋與檔案管理器降低心智負擔。

## Day 22：用命令列看颱風

原文：[命令列漫談 22](https://ithelp.ithome.com.tw/articles/10362805?sc=rss.iron)

### 核心概念

這篇把 `curl`、`jq`、ImageMagick、Ollama、TTS API 組合起來，展示如何從氣象資料 API 抓資料、處理圖片、再用 AI 摘要與語音輸出。

### 實作練習

```bash
curl -s "https://opendata.cwa.gov.tw/api/v1/rest/datastore/F-C0032-001?Authorization=$CWA_TOKEN" \
  | jq '.records.location[] | {name: .locationName, weather: .weatherElement[0].time[0].parameter.parameterName}'

magick radar.png satellite.png -compose over -composite weather.png
```

### 適合使用情境

- 把公開資料 API 轉成自己要的格式。
- 結合資料抓取、圖像處理與 AI 摘要。

### 不適合使用情境與踩雷點

- API token 不要寫死在 script 或 commit 到 repo。
- 氣象資料牽涉安全決策時，不要只靠 AI 摘要，要保留官方來源與時間。

### Junior 常見誤解

AI 可以幫忙摘要，但資料取得、欄位理解、來源可信度仍然是工程師的責任。

## Day 23：xonsh，使用 Python 語法的 Shell

原文：[命令列漫談 23](https://ithelp.ithome.com.tw/articles/10363471?sc=rss.iron)

### 核心概念

xonsh 是把 Python 語法與 shell 指令結合的 shell。它適合喜歡 Python、又想在 shell 中直接使用 Python 資料結構的人。

### 實作練習

```xsh
$PATH
files = $(ls)
len(files)
import json
data = json.loads($(cat data.json))
```

### 適合使用情境

- Python 使用者想把 shell 操作和 Python 邏輯放在一起。
- 資料處理需要 Python 套件輔助。

### 不適合使用情境與踩雷點

- 團隊共享 script 時，要確認大家有 xonsh。
- Python 模式與 shell 模式切換可能造成初學混淆。

### Junior 常見誤解

「像 Python」不代表完全沒有 shell 的複雜性。process、pipe、環境變數仍然存在。

## Day 24：新手看 Elvish 上篇

原文：[命令列漫談 24](https://ithelp.ithome.com.tw/articles/10363772?sc=rss.iron)

### 核心概念

Elvish 是現代 shell，重視結構化資料、語法一致性與互動體驗。上篇著重數學運算、JSON 處理、資料型態與變數。

### 實作練習

```elvish
+ 1 2
put [a b c]
var name = "cli"
echo $name
```

### 適合使用情境

- 想嘗試比 Bash 更現代的 shell。
- 希望 shell 有更清楚的型別與資料結構。

### 不適合使用情境與踩雷點

- 不要期待 Bash script 可直接相容。
- 新工具社群與文件量可能不如 Bash 豐富。

### Junior 常見誤解

Shell 的設計可以很多樣。Bash 是主流，不代表它是唯一合理的互動模型。

## Day 25：新手看 Elvish 下篇

原文：[命令列漫談 25](https://ithelp.ithome.com.tw/articles/10364235?sc=rss.iron)

### 核心概念

下篇延伸 Elvish 的語法、檔案預覽、命令查找、help 系統、腳本編寫與 job control，並和 Nushell、xonsh 比較。

### 實作練習

```elvish
fn greet [name]{
  echo "hello "$name
}
greet world

edit:completion:arg-completer[mycmd] = {|@words| put option-a option-b }
```

### 適合使用情境

- 想研究不同 shell 的設計取捨。
- 需要更好的互動式補全與腳本語法。

### 不適合使用情境與踩雷點

- 新 shell 不一定適合當團隊標準 shell。
- 學新 shell 前先想清楚要解決的痛點。

### Junior 常見誤解

工具選擇不是信仰題。你可以在日常用 Bash，資料探索用 Nushell，Python 工作流用 xonsh。

## Day 26：bash history 的進階用法

原文：[命令列漫談 26](https://ithelp.ithome.com.tw/articles/10364484?sc=rss.iron)

### 核心概念

Bash history 可以重複、搜尋、取上一個指令參數、預覽與編輯歷史指令。常見技巧有 `!!`、`!string`、`!$`、`!^`、`:p`、`Ctrl+R`、`fc`。

### 實作練習

```bash
echo first second third
echo !$
echo !^
history | tail
!echo:p
fc
```

### 適合使用情境

- 重跑上一個長指令。
- 快速拿上一個指令的檔名或參數。
- 找以前跑過的部署或查詢指令。

### 不適合使用情境與踩雷點

- 危險指令不要直接用 history expansion 重跑，先加 `:p` 預覽。
- history 可能存敏感資訊，避免在指令列輸入密碼或 token。

### Junior 常見誤解

history 是生產力工具，也是風險來源。它會記住你的方便，也可能記住你的秘密。

## Day 27：DuckDB 和 ClickHouse local

原文：[命令列漫談 27](https://ithelp.ithome.com.tw/articles/10365189?sc=rss.iron)

### 核心概念

DuckDB 和 ClickHouse local 都能在本機命令列用 SQL 處理資料檔，特別是 CSV、Parquet 等分析型資料。

### 實作練習

```bash
duckdb -c "SELECT count(*) FROM 'data.parquet';"
duckdb -c "SELECT status, count(*) FROM 'data.parquet' GROUP BY status;"

clickhouse local --query "SELECT status, count() FROM file('data.parquet') GROUP BY status"
```

### 適合使用情境

- 不想架資料庫，但想用 SQL 查大檔案。
- 處理 Parquet、CSV、資料分析中繼檔。

### 不適合使用情境與踩雷點

- 長期多人寫入資料庫不是它們的主要場景。
- SQL 方言與函數差異要確認，不要假設完全相容。

### Junior 常見誤解

SQL 不只存在資料庫伺服器裡。現在很多工具都能讓你直接對檔案下 SQL。

## Day 28：命令列網路資源、書、遊戲、軟體列表

原文：[命令列漫談 28](https://ithelp.ithome.com.tw/articles/10365341?sc=rss.iron)

### 核心概念

這篇整理學習資源：Data Science at the Command Line、The Linux Command Line、Command Line Reddit、OverTheWire Bandit、MIT Terminus、CLI/TUI 工具列表、CLI 遊戲與辦公工具。

### 實作練習

```bash
# Bandit 類型練習的核心是 SSH 登入與讀檔
ssh bandit0@bandit.labs.overthewire.org -p 2220
ls
cat readme
```

### 適合使用情境

- 想透過遊戲學 Linux。
- 想找 CLI 工具清單拓展視野。
- 想系統化閱讀命令列書籍。

### 不適合使用情境與踩雷點

- 資源太多時不要一次全學。先挑一條主線，例如「檔案處理」或「資料分析」。
- 練習平台的技巧不要直接套到真實系統，真實系統有權限與合規問題。

### Junior 常見誤解

學命令列不是背百科。有效方法是用一個任務驅動一組工具，反覆練習。

## Day 29：命令列增強工具 x-cmd

原文：[命令列漫談 29](https://ithelp.ithome.com.tw/articles/10366149?sc=rss.iron)

### 核心概念

x-cmd 是命令列增強工具，整合文件管理、系統監控、AI、環境管理等能力，有點像把多種 CLI 生態整合成一個入口。

### 實作練習

```bash
# 安裝前請先閱讀官方文件與安裝 script
# 不要直接執行陌生網站提供的 curl | sh
curl -fsSL https://example.com/install.sh -o install.sh
less install.sh
bash install.sh
```

### 適合使用情境

- 想探索一站式 CLI 增強工具。
- 希望快速接觸多種工具模組。

### 不適合使用情境與踩雷點

- 對安全性要求高的環境，不要未審查就安裝大型工具套件。
- `curl | sh` 很方便，但也是供應鏈風險入口。

### Junior 常見誤解

工具越大不一定越好。整合型工具要特別注意來源、更新、權限與可移除性。

## Day 30：在終端機裡面簡報

原文：[命令列漫談 30](https://ithelp.ithome.com.tw/articles/10366582?sc=rss.iron)

### 核心概念

終端機簡報工具可以用 Markdown、程式碼、圖片、動畫在 terminal 中做展示。文章提到 Slides、Lookatme、Presenterm、Patat、MDP、Pysentation、Present、Asciimatics、Terminal Text Effects。

### 實作練習

```bash
cat > slides.md <<'EOF'
# CLI Demo

---

```bash
echo hello
```
EOF

presenterm slides.md
```

### 適合使用情境

- 技術分享以 live coding 為主。
- Demo 內容本來就在 terminal。
- 想把投影片納入版本控制。

### 不適合使用情境與踩雷點

- 大型正式簡報仍要測投影環境、字型、解析度與圖片支援。
- 不要在簡報中執行不可逆指令。

### Junior 常見誤解

簡報工具不是重點，重點是內容能不能穩定展示。terminal 簡報很帥，但事前彩排更重要。

## Day 31：讓 TTE 支援中文的文字特效

原文：[命令列漫談 31](https://ithelp.ithome.com.tw/articles/10366856?sc=rss.iron)

### 核心概念

Terminal Text Effects 原本處理中文時會遇到寬度、殘影、線條或形狀錯位問題。作者展示修正後的版本，讓中文在 SynthGrid、Spotlight、Beams、Fireworks、Waves、Decrypt 等效果中能更正常顯示。

### 實作練習

```bash
python -m venv .venv
source .venv/bin/activate
pip install terminaltexteffects
tte decrypt --text "命令列也可以很有趣"
```

### 適合使用情境

- 終端機展示、教學、簡報開場。
- 想研究 Unicode、中文字寬、terminal rendering。
- 做 CLI 工具時需要正確處理 CJK 字元。

### 不適合使用情境與踩雷點

- 文字特效不應影響核心資訊可讀性。
- 中文寬度處理不能只用字元數，要考慮 display width。

### Junior 常見誤解

字串長度不等於畫面寬度。英文、中文、emoji、組合字元在 terminal 中的顯示寬度可能不同。

## 實務總結：命令列能力的三層

### 第一層：會查

你能用 `ls`、`cat`、`head`、`tail`、`grep`、`find`、`rg`、`ps`、`df`、`ss` 找到資訊。

### 第二層：會串

你能把 `curl`、`jq`、`sort`、`uniq`、`tee`、`xargs` 串起來，變成可重複的資料流程。

### 第三層：會判斷

你知道什麼時候用 Bash、什麼時候換 Python、什麼時候用 DuckDB、什麼時候不該在 production 直接跑指令。

## 建議練習順序

1. 每天挑一篇，先照著「實作練習」打一次。
2. 把輸出導到檔案，例如 `> output.txt` 或 `| tee output.txt`。
3. 故意改錯一個參數，觀察錯誤訊息。
4. 把同一個任務分別用傳統工具、Python、jq 或 DuckDB 做一次。
5. 最後把常用流程整理成 `scripts/` 裡的 shell script。

## 最後提醒

- 任何破壞性指令，例如 `rm`、`mv`、`chmod`、`chown`、`kill`、`docker rm`，先確認目標。
- 任何會把資料送到第三方 API 的流程，先確認資料是否包含個資、token、公司機密。
- 任何 `curl | sh` 安裝方式，先下載、閱讀，再決定是否執行。
- 命令列的價值不是炫技，而是讓工作更可重複、更可檢查、更可自動化。
