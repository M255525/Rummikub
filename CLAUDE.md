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

## 內建的遊戲規則（改動需一致）

牌組 106 張（紅／藍／橘／黑 四色 ×1–13 各兩組 = 104，＋百搭牌 2 張）；合法牌型為 Group（3–4 張同數字不同色）與 Run（3 張以上同色連號）；首次出牌新放牌總分需 ≥30；電腦對手 1–3 位。AI 為基礎策略——不會重排牌桌、手中百搭牌不會主動使用。

## 中高齡友善設計（編輯 UI 時務必維持）

這些不是可有可無的樣式，而是本專案的核心約束：文字最小 18px（放大模式 22px）、按鈕點擊區 ≥44×44px、每個操作都有中文文字說明而非純圖示、不設讀秒倒數、無效出牌即時以紅框＋文字提示原因、可一鍵取消整回合操作。另有高對比模式與 RWD 斷點（手機 <600px／平板 600–980px／桌機 >980px），同時支援滑鼠拖拉與觸控點選。變更版面時請沿用上述規格。

## 頂部跑馬燈與「關於」資訊（2026-08-05 新增）

`#marqueeBar` 顯示跟工作區其他工具（`expense-tracker-pwa`、`ai-video-studio` 主版等）共用同一份 Google Sheet 維護的公告內容，同一個授權伺服器 Apps Script 網址。這個遊戲**沒有序號登入機制**，做法是頁面載入時直接 POST 空序號給該網址，只取回傳的 `marquee` 欄位（忽略 `valid`/`reason`）。`localStorage` key 為 `rmkMarquee`，先讀快取立即顯示、再背景 fetch，每 20 分鐘重抓一次；抓取失敗靜默忽略，**不影響「無需網路即可開玩」的核心賣點**（README 明確標榜的特性）。

**版面整合方式**：`html,body{height:100%;overflow:hidden}` ＋ `body{display:flex;flex-direction:column}`——這個殼層自己管理整個視窗高度，所以 `#marqueeBar` 是插進 `<body>` 最上面的一個普通 flex item（`flex-shrink:0`），不是 `position:fixed`。`#overlay`／`#tut-overlay`／`#settings` 都是 `position:fixed`，不受 flex 排列順序影響，DOM 位置可以不用調整。

使用警語（僅供個人娛樂與教學示範使用）＋「創作者：蔡豐全（Mark Tsai）」放在「⚙ 設定」面板最下面（`#settings` 內），不做成常駐頁尾——遊戲畫面寸土寸金，這裡沒有另外的「設定頁」可去，直接用既有的設定面板當作「關於」資訊的落點。

**`拉密Rummikub-GoogleSites嵌入用.html` 已同步套用**（兩個檔案本來就要求保持同步，見上方「架構」一節）。這個嵌入版本除了少了 `<!DOCTYPE>`／`<html>`／`<head>`／`<body>` 這幾行包裝標籤之外，內容與主檔逐行相同；之後任何改動都可以用這個做法重新產生嵌入版，不需要手動逐一對照：

```python
import io
with io.open('拉密Rummikub.html', encoding='utf-8', newline='') as f:
    lines = f.readlines()
# 移除 DOCTYPE/html/head/title 開頭六行、</head>+<body> 兩行、</body>+</html> 結尾兩行
# （實際行號請對照當下檔案的 </head>/<body>/</body>/</html> 位置）
```

**踩坑**：用 Python 重新產生嵌入版檔案時，`io.open(..., 'w', encoding='utf-8')` 若不指定 `newline=''`，Windows 上會把原本的 LF 換行自動轉成 CRLF，導致 `git diff` 整個檔案看起來每一行都變了（明明內容沒變）。務必加 `newline=''` 保持原始換行符不被轉換。
