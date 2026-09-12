# NTHU Live AI 安裝與部署手冊

## 先看這裡：只需要完成 6 步

如果你只想讓 Live AI 跑起來，不需要先安裝所有進階模型。請依序完成：

1. 安裝 NVIDIA 顯示卡驅動，重新開機。
2. 安裝 Anaconda。
3. 建立 `livetalking` Python 3.12 環境。
4. 安裝專案 Python 套件與前端套件。
5. 確認 GPU 顯示 `CUDA: True`。
6. 雙擊 `start-mira-live.cmd`，開啟 `http://localhost:3001/`。

### 必須安裝的軟體

| 軟體 | 用途 | 下載／說明 |
|---|---|---|
| NVIDIA Driver | 讓 Python 使用顯示卡 | [NVIDIA 官方驅動下載](https://www.nvidia.com/Download/index.aspx) |
| Anaconda | 建立 `livetalking` 環境 | [Anaconda 官方下載](https://www.anaconda.com/download) |
| Git | 取得專案原始碼 | [Git for Windows](https://git-scm.com/download/win) |
| cloudflared（只有要公開網址才需要） | Cloudflare Tunnel | [Windows 下載與安裝](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/downloads/#windows) |

### 不需要另外安裝的項目

- 不需要另外安裝 CUDA Toolkit，先使用專案環境內的 CUDA / PyTorch。
- 不需要先安裝 ChatGPT 或 Gemini；它們是選用的雲端 AI。
- 不需要先安裝 HeyGen；它是選用的付費雲端服務。

### 每一步應該看到的畫面

| 步驟 | 成功畫面 |
|---|---|
| GPU 驅動 | `nvidia-smi` 顯示 GPU 型號與 Driver Version |
| Anaconda 環境 | 命令列開頭出現 `(livetalking)` |
| CUDA 檢查 | 顯示 `CUDA: True` 與 GPU 名稱 |
| 啟動服務 | 視窗顯示 `ready: http://localhost:3001/` |
| 網站 | 顯示 NTHU 標誌、角色主畫面與「開始 LIVE」按鈕 |

看到其中一個步驟失敗時，先不要繼續，直接回到該步驟的「常見問題」段落。

本手冊說明如何在 Windows 主機安裝、啟動與測試 NTHU Live AI。專案路徑假設為：

```text
C:\Users\user\Documents\Codex\vTuber
```

系統由兩個本機服務組成：

| 元件 | 位址 | 用途 |
|---|---|---|
| 前端 | `http://localhost:3001` | 角色、背景、字幕與操作介面 |
| LiveTalking API | `http://127.0.0.1:8010` | 語音、對嘴、影片生成與 Live 連線 |

## 1. 環境需求

- Windows 10/11 64-bit
- NVIDIA 顯示卡與最新驅動程式
- CUDA 可用，建議使用已安裝的 CUDA / PyTorch GPU 環境
- Anaconda 或 Miniconda
- Node.js（專案使用 Codex bundled Node runtime，也可使用相容的 Node 20+）
- 至少 50 GB 可用磁碟空間，模型與快取會佔用大量空間

確認 NVIDIA 驅動：

```powershell
nvidia-smi
```

## 2. 取得專案

```powershell
cd "$HOME\Documents\Codex"
git clone https://github.com/lipku/livetalking.git vTuber
cd .\vTuber
```

若專案已存在，直接進入專案目錄即可。不要重複 clone 到另一個資料夾，避免啟動到舊版本。

## 3. 建立 Anaconda 環境

```powershell
conda create -n livetalking python=3.12 -y
conda activate livetalking
python --version
```

應顯示 Python 3.12.x。

安裝專案 Python 套件：

```powershell
cd "$HOME\Documents\Codex\vTuber\work\LiveTalking"
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

若專案提供模型專用 requirements，依序安裝對應檔案。PyTorch 必須安裝 CUDA 版本，不要使用 CPU-only 版本。

確認 GPU 可被 PyTorch 使用：

```powershell
python -c "import torch; print('CUDA:', torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU')"
```

`CUDA: True` 才表示後端會使用 GPU。

## 4. 安裝前端套件

```powershell
cd "$HOME\Documents\Codex\vTuber"
npm install
```

若使用專案提供的 bundled Node，請確認 `start-mira-live.cmd` 中的 Node 路徑仍然存在。

## 5. 模型與資料

模型檔案應放在專案的 `work` 目錄及 LiveTalking 所要求的子目錄。常見模型包括：

- MuseTalk：即時對嘴模型
- Ditto：高品質數字人模型
- LTX：文字轉高畫質影片模型，單段語音建議控制在 15 秒內
- LeapTalk Pro：若已安裝，需確認其 Python 環境與 GPU 相容
- HeyGen：雲端 API，需由使用者自行輸入 API key

不要把 API key、Tunnel token 或其他密鑰寫入 Git。建議使用 `.env` 或 `.env.heygen.local`，並確認它已列入 `.gitignore`。

## 6. 啟動本機 Live AI

建議使用專案提供的啟動檔：

```powershell
cd "$HOME\Documents\Codex\vTuber"
Start-Process .\start-mira-live.cmd
```

啟動檔會：

1. 啟動 GPU 後端，監聽 `8010`
2. 啟動前端，監聽 `3001`
3. 等待兩個服務就緒

瀏覽器開啟：

```text
http://localhost:3001/
```

## 7. 啟動狀態檢查

```powershell
Invoke-WebRequest http://localhost:3001/ -UseBasicParsing
Invoke-WebRequest http://127.0.0.1:8010/api/latentsync/models -UseBasicParsing
Get-NetTCPConnection -LocalPort 3001,8010 -State Listen
```

前兩個請求應回應 `200`。若後端根目錄回應 `403`，不代表故障，請改測試 `/api/latentsync/models`。

## 8. 前端連線設定

本機使用時，前端「伺服器位址」設定為：

```text
http://127.0.0.1:8010
```

公開測試時，這個欄位必須改成後端的公開 HTTPS 網址，不能保留 `127.0.0.1`，因為外部使用者的瀏覽器會把它當成自己的電腦。

### 網址對照表

| 情境 | 前端網址 | 伺服器位址 |
|---|---|---|
| 本機測試 | `http://localhost:3001/` | `http://127.0.0.1:8010` |
| Quick Tunnel | `https://<前端隨機名稱>.trycloudflare.com` | `https://<後端隨機名稱>.trycloudflare.com` |
| 固定網域 | `https://nthuai96.edu.nthu.edu.tw` | `https://api-live.nthuai96.edu.nthu.tw`（需另建 API 路由） |

### 前端畫面設定位置

1. 開啟前端網址。
2. 向下捲動至「塑造你的數字人」區域。
3. 切換至「LiveTalking」分類。
4. 找到「伺服器位址」欄位。
5. 輸入後端網址，按「儲存連線設定」。
6. 重新整理頁面，再按「開始 LIVE」。

畫面中可確認：左上顯示「本機服務」或公開 HTTPS 網址、角色影像位於中央、下方顯示「自然 · LTX 高品質 · 1024×576 · 非即時」，以及對話輸入框與男女聲音按鈕。

## 9. Cloudflare Tunnel 測試

### 9.1 前端臨時網址

開啟 PowerShell：

```powershell
& "C:\Program Files (x86)\cloudflared\cloudflared.exe" tunnel --url http://localhost:3001
```

Cloudflare 會顯示一個 `https://xxxxx.trycloudflare.com` 網址。保持視窗開啟。

### 9.2 後端臨時網址

再開另一個 PowerShell：

```powershell
& "C:\Program Files (x86)\cloudflared\cloudflared.exe" tunnel --url http://localhost:8010
```

把第二個 `trycloudflare.com` 網址填入前端「伺服器位址」，儲存後重新整理。

測試後端：

```powershell
Invoke-WebRequest https://你的後端網址.trycloudflare.com/api/latentsync/models -UseBasicParsing
```

Quick Tunnel 適合短期測試，網址會變動，也不適合正式服務。正式服務請使用 Named Tunnel。[Cloudflare Tunnel 官方文件](https://developers.cloudflare.com/tunnel/get-started/)

### Quick Tunnel 成功畫面判讀

終端機成功時會顯示：

```text
Your quick Tunnel has been created! Visit it at:
https://<隨機名稱>.trycloudflare.com
Registered tunnel connection
```

瀏覽器成功時，首頁應顯示 NTHU 國立清華大學標誌、角色主畫面、字幕區、聲音選擇及文字輸入框。若首頁能開啟但按下送出後顯示 `failed to fetch`，代表前端的「伺服器位址」尚未改成後端 Tunnel 網址。

### 9.3 Windows 服務

正式 Tunnel 可安裝為 Windows 服務。請以系統管理員身分開啟 PowerShell：

```powershell
cloudflared.exe service install <TUNNEL_TOKEN>
Get-Service cloudflared
```

狀態應為 `Running`。若出現 `Access is denied`，表示 PowerShell 沒有用系統管理員權限。

### 9.4 固定網域

Cloudflare 後台設定：

```text
Networking → Tunnels → 選取 Tunnel → Routes → Add route
Hostname: nthuai96.edu.nthu.edu.tw
Service: http://localhost:3001
```

DNS 管理員需新增：

```text
類型：CNAME
名稱：nthuai96
內容：<TUNNEL-ID>.cfargotunnel.com
Proxy：開啟
```

若 DNS 不在你的管理權限內，必須請網管代為新增。固定 IP 不會自動等於 Cloudflare Tunnel。

## 10. 常見問題

### `failed to fetch`

檢查前端「伺服器位址」是否仍為 `127.0.0.1:8010`。公開網址必須填入後端 Tunnel 的 HTTPS 網址，並確認後端 Tunnel 視窗仍在執行。

### 網站可開啟，但 Live 沒反應

```powershell
Invoke-WebRequest http://127.0.0.1:8010/api/latentsync/models -UseBasicParsing
```

若不是 `200`，重新啟動後端。再檢查 `nvidia-smi` 是否看得到 Python GPU 使用量。

### GPU 沒有被使用

```powershell
conda activate livetalking
python -c "import torch; print(torch.version.cuda); print(torch.cuda.is_available())"
```

若 `False`，需重新安裝相容的 CUDA 版 PyTorch、確認 NVIDIA 驅動，並避免啟動到其他 Python 環境。

### `403 Forbidden` 出現在 `http://127.0.0.1:8010/`

後端根路徑受保護或未提供首頁。請使用健康檢查 API：

```text
http://127.0.0.1:8010/api/latentsync/models
```

### Cloudflare 網址失效

Quick Tunnel 關閉終端機就會失效，重新執行指令會得到新網址。正式環境應使用 Named Tunnel 與固定 DNS 路由。

## 11. 停止服務

關閉啟動的前端、後端與 Quick Tunnel 視窗，或使用：

```powershell
Get-NetTCPConnection -LocalPort 3001,8010 -State Listen | Select-Object OwningProcess
```

確認 PID 後再停止對應程序。不要停止不相關的 Python 程序。

## 12. 驗收清單

- [ ] `http://localhost:3001/` 可開啟
- [ ] `/api/latentsync/models` 回應 200
- [ ] `torch.cuda.is_available()` 為 `True`
- [ ] MuseTalk 可產生語音與對嘴畫面
- [ ] 字幕可完整顯示繁體中文
- [ ] 前端背景切換會更新主畫面
- [ ] 文字轉影片可選擇是否生成字幕
- [ ] 公開測試時前端與後端都使用 HTTPS Tunnel 網址
- [ ] 固定網域 DNS 已由管理員建立 CNAME
