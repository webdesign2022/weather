🍌 NANO BANANA 技術規格與開發手冊 (Technical Specification)
版本： V10.2 (Cloud Sync & ID Reveal)
最後更新日期： 2024/05
適用對象： 前端工程師、產品經理、系統維護人員

1. 專案概述 (Project Overview)
NANO BANANA 是一個基於 Web 技術的「全息投影 (Hologram) 禮物系統」。核心商業模式為「舊機利用」與「情感連結」。
    • 顯示端 (Display Client)： 將閒置舊手機放入金字塔投影盒，作為全息顯示器（電子相框/天氣站）。
    • 遙控端 (Remote Client)： 使用主手機透過網頁遠端控制顯示內容、發送訊息、更換主題。
    • 核心價值： 環保（舊機再造）、儀式感（全息視覺）、即時互動（雲端控制）。

2. 系統架構 (System Architecture)
本專案採用 Serverless (無伺服器) 架構，前端直接與 Firebase 溝通，大幅降低維運成本。
2.1 架構圖
    • Client A (顯示端): 純前端渲染 (Video/DOM)，監聽 Firebase 數據變化。
    • Client B (遙控端): 控制介面，寫入數據至 Firebase。
    • Backend (BaaS): Google Firebase Realtime Database (儲存狀態、訊息、用戶關聯)。
    • External API: OpenWeatherMap API (獲取即時天氣)。
2.2 技術棧 (Tech Stack)
    • 前端核心: HTML5, CSS3, Vanilla JavaScript (無須編譯，輕量化)。
    • 資料庫: Firebase Realtime Database (WebSocket 即時連線)。
    • 硬體存取:
        ◦ Screen Wake Lock API: 防止顯示端螢幕休眠。
        ◦ Geolocation API: 獲取天氣定位。
        ◦ localStorage: 本地緩存用戶身分與裝置列表。
    • 部署形式: 靜態網頁 (GitHub Pages / Firebase Hosting / Vercel)。
    • PWA: 支援 Add to Home Screen，模擬原生 App 體驗。

3. 資料庫設計 (Database Schema)
Firebase NoSQL JSON 結構設計如下：
3.1 devices/ (裝置狀態節點)
儲存每一台「顯示端」的即時狀態。以 Device ID (例如 A1B2) 為 Key。
JSON
{
  "devices": {
    "A1B2": {
      "theme": "weather",         // 當前主題 (weather, jellyfish, rain, cyber)
      "message": {
        "text": "生日快樂!",       // 彈幕文字
        "timestamp": 1715829399   // 時間戳 (用於判斷是否為新訊息)
      },
      "settings": {
        "recipient": "Amy",       // 收禮者暱稱
        "birthday": "1995-05-20", // 生日
        "pairedBy": "0912345678"  // 配對者的手機號 (Owner)
      }
    }
  }
}
3.2 users/ (用戶歸戶節點) (V10.0 新增)
儲存「遙控端」用戶擁有的裝置列表，實現換機同步。以 手機號碼 為 Key。
JSON
{
  "users": {
    "0912345678": {
      "devices": [
        { "id": "A1B2", "name": "Amy的禮物" },
        { "id": "X7Z9", "name": "客廳展示機" }
      ]
    }
  }
}

4. 核心功能流程 (Key Workflows)
4.1 裝置配對流程 (Pairing)
    1. 顯示端： 初始化 -> 產生 4 碼亂數 ID (如 A1B2) -> 寫入 localStorage -> 監聽 DB devices/A1B2。
    2. 遙控端： 輸入 A1B2 -> 輸入送禮資訊 -> 寫入 DB devices/A1B2/settings。
    3. 顯示端： 偵測到 settings 被建立 -> 隱藏 ID 畫面 -> 進入天氣主題。
    4. 雲端備份： 遙控端將 {id: "A1B2", name: "..."} 寫入 users/手機號/devices。
4.2 身分恢復與防呆 (Recovery & Safety)
    • 顯示端救援： 若清除快取，使用者可點擊「我已有 ID」-> 輸入舊 ID -> 恢復連線。
    • ID 查詢 (V10.2)： 遙控端介面下方恆顯「裝置 ID (黃字)」，方便用戶查看。
    • ID 喚醒 (V10.1)： 顯示端運行中點擊螢幕 -> 浮現巨大 ID 5秒 -> 自動隱藏 (防止忘記 ID)。
4.3 雲端同步 (Cloud Sync)
    • 登入： 用戶輸入手機號碼。
    • 同步： 系統讀取 users/手機號 -> 將裝置列表寫入遙控端 localStorage。
    • 結果： 換手機、重裝瀏覽器，裝置列表自動回來。

5. 檔案結構與模組說明 (Directory Structure)
目前為 MVP (Minimum Viable Product) 單檔架構，未來建議拆分。
當前結構 (Current)
/root
  ├── index.html       # 包含 HTML, CSS, JS 的核心單一文件
  ├── admin.html       # (選配) 管理者後台
  ├── manifest.json    # (建議新增) PWA 設定檔
  └── assets/          # 媒體資源
      ├── weather_clear.mp4
      ├── weather_rain.mp4
      ├── theme_jelly.mp4
      └── fonts/
建議擴充結構 (Future Refactoring)
當專案變大時，應拆分如下：
/src
  ├── css/
  │   ├── main.css      # 全局樣式
  │   └── themes.css    # 主題特效樣式
  ├── js/
  │   ├── app.js        # 入口路由 (Role Router)
  │   ├── firebase.js   # DB 初始化與封裝
  │   ├── controller.js # 遙控端邏輯 (Remote Logic)
  │   └── display.js    # 顯示端邏輯 (Display Logic)
  └── index.html

6. 開發歷程日誌 (Development Log)
記錄每個版本的關鍵迭代，方便追溯功能演進。
版本	功能代號	關鍵更新內容
V1-V4	Prototype	基礎 HTML 結構，金字塔全息 CSS 變形 (transform: rotate)，影片背景。
V5-V8	IoT Core	導入 Firebase，實現手機遙控切換影片 (Weather/Jellyfish)。
V9.0	UX Polish	加入天氣 API 真實數據，增加 Toast 訊息傳送功能。
V9.5	Recovery	新增「恢復舊 ID」功能，加入 Wake Lock 防止螢幕休眠。
V10.0	Cloud Sync	導入「手機號碼帳號體系」，實現裝置列表雲端備份與跨裝置同步。
V10.1	ID Reveal	顯示端新增「點擊螢幕喚醒 ID」防呆機制。
V10.2	Visual ID	遙控端新增「常駐顯示裝置 ID」，方便用戶查閱抄寫。

7. 介面與使用者體驗設計 (UI/UX)
7.1 全息顯示端 (Display)
    • 風格： 黑色背景 (Pure Black #000000)，高對比螢光色 (Cyberpunk)。
    • 布局： 十字型排列 (上/下/左/右)，中心留空放置金字塔尖端。
    • 字體： Orbitron (科幻數字感) + Roboto。
7.2 遙控端 (Remote)
    • 風格： 深色模式控制台，卡片式設計 (Card UI)。
    • 交互：
        ◦ 即時反饋： 按下按鈕 -> Toast 提示 "已發送"。
        ◦ 狀態同步： 下拉選單切換裝置，下方資訊欄即時更新。

8. 未來擴充建議 (Roadmap)
若要繼續商業化，建議開發以下功能：
    1. Native App (iOS/Android):
        ◦ 使用 Flutter 或 React Native 打包，解決瀏覽器網址列佔用空間的問題，並獲得更穩定的 Wake Lock 權限。
    2. 商城系統 (Theme Store):
        ◦ 目前的「主題切換」是免費的。未來可做成「付費解鎖」精美 3D 動畫主題 (例如：生日蛋糕、漫威英雄)。
    3. 定時任務 (Scheduling):
        ◦ 設定「早上 7:00 自動播放鬧鐘主題」、「晚上 10:00 自動變暗」。
    4. 語音控制:
        ◦ 整合 Web Speech API，對著舊手機喊「下雨了」，直接切換雨天場景。

📝 給開發人員的一句話
「NANO BANANA 的代碼哲學是 K.I.S.S. (Keep It Simple, Stupid)。我們不依賴複雜的 Framework (如 React/Vue)，而是用最原生的 Web 技術，讓任何一支 5 年前的舊安卓手機都能流暢運行。擴充功能時，請務必考慮舊手機的效能負擔。」
