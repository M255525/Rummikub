# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

以中高齡使用者為設計核心的拉密（Rummikub）單機桌遊。純前端、單一檔案、無外部相依、無伺服器，開啟即玩。

## 架構

- **`拉密Rummikub.html`** — 遊戲本體。**所有 HTML／CSS／JavaScript 與遊戲邏輯都在這一個檔案內**，沒有建置步驟、沒有套件、沒有測試。直接以瀏覽器開啟即可執行；修改後重新整理瀏覽器即生效。
- **`拉密Rummikub-GoogleSites嵌入用.html`** — 供 Google Sites 內嵌的變體版本。
- **`一鍵複製-貼到GoogleSites.bat`** — 把嵌入版 HTML 內容複製到剪貼簿，方便貼進 Google Sites。
- **`index.html`** — GitHub Pages 用的重新導向 stub（`meta refresh` 導到 `拉密Rummikub.html`），不是遊戲本體，內容不需要跟著遊戲邏輯同步更新。

**兩個 HTML 版本需保持同步**：修改遊戲邏輯或介面時，主檔與 Google Sites 嵌入版都要一起更新（或由主檔重新產生嵌入版），否則兩處行為會分歧。

**GitHub Pages 上線網址**：<https://m255525.github.io/Rummikub/>（2026-08-05 啟用，`master` 分支根目錄）。公開 repo：<https://github.com/M255525/Rummikub>。因為遊戲本體檔名是中文（`拉密Rummikub.html`），GitHub Pages 預設只認 `index.html` 當首頁，所以另外放了一個極簡的 `index.html` 導向頁，避免把遊戲內容複製成兩份要同步。**這個上線管道跟 Google Sites 是分開的兩條路**：`git push` 到這個 repo 後 GitHub Pages 會自動重新部署（通常一兩分鐘內生效）；Google Sites 版本仍然要手動雙擊 `一鍵複製-貼到GoogleSites.bat` 重新貼上發布，兩邊不會互相同步，改完程式碼要記得兩邊都處理。

## 加入主畫面（PWA，2026-08-14 新增）

比照工作區 `expense-tracker-pwa` 的做法：`manifest.json`＋`icons/`（棋盤綠 `#1e5e1e` 背景、金色 `#FFD700`「拉」字圖示）＋`service-worker.js`（network-first＋同源快取備援，不需要每次改動升版 `CACHE_NAME`）。**manifest／SW 只掛在遊戲本體 `拉密Rummikub.html`，不動 `index.html`（純轉址 stub，不需要）與 `拉密Rummikub-GoogleSites嵌入用.html`（貼進 Google Sites 的內嵌內容，不是獨立頁面載入，無法安裝也不該註冊 SW）**。安裝按鈕（`#installBtn`，📲 加入主畫面）放在 `#settings` 面板內、「🔄 重新開始」下方，沿用既有 `.hbtn` 樣式。已用 Playwright 實測 Chromium 觸發 `beforeinstallprompt`、SW 註冊成功。


**iOS／iPadOS／macOS 相容性補強（2026-08-14 同日追加）**：Safari（含 iOS 上的 Chrome/Firefox，底層都是 WebKit）**永遠不會觸發 `beforeinstallprompt`**，原本的按鈕邏輯在這些瀏覽器上一律落入「瀏覽器不支援」這句話，其實是誤導——蘋果裝置本來就能加入主畫面，只是要透過分享選單手動操作，不像 Chrome/Edge 有自動彈窗。修法：安裝腳本新增 `isIOSDevice`（`/iPad|iPhone|iPod/` 或 `navigator.platform==='MacIntel' && maxTouchPoints>1`——後者是因為 iPadOS 13+ 預設偽裝成 Mac 桌面版 UA，要用觸控點數才分得出來是 iPad 還是真的 Mac）與 `isMacDesktop && isSafariEngine`（macOS 桌面版 Safari 17+ 是「檔案→加入 Dock」，跟手機的分享選單操作不同）兩種判斷，各自顯示對應的操作指引文字，不再顯示「不支援」；`isStandalone`（`matchMedia('(display-mode: standalone)')` 或 iOS 專有的 `navigator.standalone`）為真時直接隱藏按鈕（已經是安裝後開啟，不需要再顯示安裝按鈕）。`<head>` 同步補上 `apple-touch-icon`（180×180 專用尺寸，`icons/apple-touch-icon.png`，純色不透明背景）＋ `apple-mobile-web-app-capable`／`mobile-web-app-capable`（兩個都要，前者給 Safari、後者是 Chrome 主張的新標準，只寫一個 Chrome 會在主控台噴 deprecation warning）＋ `apple-mobile-web-app-status-bar-style`／`apple-mobile-web-app-title`。這五個判斷/訊息字串在全部 9 個已加裝 PWA 的專案裡是逐字複製的同一段邏輯，日後若要調整任一處的措辭或判斷式，建議九個一起改，避免各專案的安裝體驗不一致。

## 內建的遊戲規則（改動需一致）

牌組 106 張（紅／藍／橘／黑 四色 ×1–13 各兩組 = 104，＋百搭牌 2 張）；合法牌型為 Group（3–4 張同數字不同色）與 Run（3 張以上同色連號）；首次出牌新放牌總分需 ≥30；電腦對手 1–3 位。AI 為基礎策略——不會重排牌桌、手中百搭牌不會主動使用。

## 中高齡友善設計（編輯 UI 時務必維持）

這些不是可有可無的樣式，而是本專案的核心約束：文字最小 18px（放大模式 22px）、按鈕點擊區 ≥44×44px、每個操作都有中文文字說明而非純圖示、不設讀秒倒數、無效出牌即時以紅框＋文字提示原因、可一鍵取消整回合操作。另有高對比模式與 RWD 斷點（手機 <600px／平板 600–980px／桌機 >980px），同時支援滑鼠拖拉與觸控點選。變更版面時請沿用上述規格。

## 頂部跑馬燈與「關於」資訊（2026-08-05 新增）

`#marqueeBar` 顯示跟工作區其他工具（`expense-tracker-pwa`、`ai-video-studio` 主版等）共用同一份 Google Sheet 維護的公告內容，同一個授權伺服器 Apps Script 網址。這個遊戲**沒有序號登入機制**，做法是頁面載入時直接 POST 空序號給該網址，只取回傳的 `marquee` 欄位（忽略 `valid`/`reason`）。`localStorage` key 為 `rmkMarquee`，先讀快取立即顯示、再背景 fetch，每 20 分鐘重抓一次；抓取失敗靜默忽略，**不影響「無需網路即可開玩」的核心賣點**（README 明確標榜的特性）。

**版面整合方式**：`html,body{height:100%;overflow:hidden}` ＋ `body{display:flex;flex-direction:column}`——這個殼層自己管理整個視窗高度，所以 `#marqueeBar` 是插進 `<body>` 最上面的一個普通 flex item（`flex-shrink:0`），不是 `position:fixed`。`#overlay`／`#tut-overlay`／`#settings` 都是 `position:fixed`，不受 flex 排列順序影響，DOM 位置可以不用調整。

使用警語（僅供個人娛樂與教學示範使用）＋「創作者：蔡豐全（Mark Tsai）」放在「⚙ 設定」面板最下面（`#settings` 內），不做成常駐頁尾——遊戲畫面寸土寸金，這裡沒有另外的「設定頁」可去，直接用既有的設定面板當作「關於」資訊的落點。

## 手機版面修正（2026-08-05）

實測手機寬度（390px／360px）發現 `#hdr` 與 `#actbar` 會把長文字（`#msg` 遊戲狀態訊息、`#turn-info` 回合提示）跟標題／按鈕擠在同一個 flex 行內：由於這兩個文字欄位是 `flex:1`，在窄螢幕上可用寬度被按鈕吃光後，文字被壓成一條極窄直欄、逐字換行成一長串，把整個 header／action bar 撐到 110px／179px 高，嚴重壓縮牌桌可視面積。

**修法**：在既有 `@media(max-width:600px)` 斷點內，給 `#msg`／`#turn-info` 加 `flex-basis:100%`（`#msg` 另加 `order:5` 讓它排到標題／按鈕後面自成一行），讓它們各自獨占一整行，標題＋按鈕、4 顆操作按鈕則維持在各自單獨一行內（實測寬度足夠容納，不會再被文字擠壓）。效果：header 110px→83px、action bar 179px→88px，牌桌可視高度增加約 120px（390×844 視窗下）。**點選（tap-to-select）出牌流程本來就是用一般 `onclick`，不依賴 HTML5 拖拉事件，觸控裝置上原本就能正常運作**——這次調整純粹是版面壓縮問題，不是互動邏輯問題。已用 Playwright 在 390px／360px 寬度下實測版面與完整選牌→加入新組→確認出牌流程都正常。

**`拉密Rummikub-GoogleSites嵌入用.html` 已同步套用**（兩個檔案本來就要求保持同步，見上方「架構」一節）。這個嵌入版本除了少了 `<!DOCTYPE>`／`<html>`／`<head>`／`<body>` 這幾行包裝標籤之外，內容與主檔逐行相同；之後任何改動都可以用這個做法重新產生嵌入版，不需要手動逐一對照：

```python
import io
with io.open('拉密Rummikub.html', encoding='utf-8', newline='') as f:
    lines = f.readlines()
# 移除 DOCTYPE/html/head/title 開頭六行、</head>+<body> 兩行、</body>+</html> 結尾兩行
# （實際行號請對照當下檔案的 </head>/<body>/</body>/</html> 位置）
```

**踩坑**：用 Python 重新產生嵌入版檔案時，`io.open(..., 'w', encoding='utf-8')` 若不指定 `newline=''`，Windows 上會把原本的 LF 換行自動轉成 CRLF，導致 `git diff` 整個檔案看起來每一行都變了（明明內容沒變）。務必加 `newline=''` 保持原始換行符不被轉換。

## 排行榜（2026-08-05 新增）

`localStorage` key `rmk-leaderboard`，本機前 5 名。**這個遊戲沒有像切水果那樣的「分數」**（是跟電腦對戰的桌遊，不是計分制），所以排名依據改用「玩家出牌回合數」（`G.playerTurns`，`doConfirm()` 成功出牌與 `doDraw()` 摸牌時各自累加一次），數字越小（越快出完手牌）排名越前面。

**只有玩家獲勝才會計入排行榜**——電腦獲勝或牌堆摸完平局時，`showOverlay()` 第三個參數 `isWin` 是 `falsy`，不會顯示暱稱輸入框，但排行榜列表仍然照樣顯示（單純不能新增）。玩家獲勝時才判斷 `G.playerTurns` 是否擠得進前 5（`lbQualifies()`：不到 5 筆或回合數少於目前第 5 名），擠得進才顯示暱稱輸入框（留白預設「訪客」），按「🏆 記錄成績」才寫入並依回合數重新排序（由小到大）、截斷到 5 筆。UI 直接整合進既有的結算 `#overlay` Modal 內（`<p id="mbody">` 跟按鈕列之間），不是另開一個畫面。

**`拉密Rummikub-GoogleSites嵌入用.html` 已同步套用**（跟手機版面修正那次一樣，用同一套 Python 腳本重新產生，記得換算當時檔案的 `</head>`/`<body>`/`</body>`/`</html>` 行號）。
