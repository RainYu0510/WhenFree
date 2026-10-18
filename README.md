# 有空嗎？ WhenFree

**喬時間，一起更簡單！**

參考 When2meet 的操作概念，讓社團、朋友或團隊透過同一個活動連結，標記各自有空的時間，再找出最多人能參加的時段。

這是一個以 Vibe Coding 開發的互動網頁：前端的 HTML、CSS、JavaScript 全部放在單一 `index.html`，透過 Firebase 提供匿名登入與即時資料同步，並使用 GitHub Pages 發布。

- 網站入口：[有空嗎？](https://rainyu.dev/WhenFree/)（正式網址以儲存庫 Settings → Pages 顯示為準）
- 原始碼：[RainYu0510/WhenFree](https://github.com/RainYu0510/WhenFree)
- 作者：余萬崧 

## 功能與規格

| 功能 | 說明 |
| --- | --- |
| 建立活動 | 輸入活動名稱，選擇最多連續 7 天的日期範圍與每日起訖時間。 |
| 連結邀請 | 複製活動專屬連結；成功後按鈕顯示「✓ 已複製」，約 2.5 秒後恢復。 |
| 代碼加入 | 在首頁輸入 4 位數活動代碼，例如 `0042`；開頭的 `0` 必須保留。 |
| 舊活動補發代碼 | 原建立者可在尚未取得代碼的活動中按「產生代碼」。 |
| 填寫時間 | 每格為 30 分鐘；滑鼠可點擊或拖曳，手機可輕點選取並滑動瀏覽。 |
| 修改紀錄 | 同一個瀏覽器身分可重新開啟活動並修改自己的名字與時間。 |
| 即時統計 | 「大家的時間」以顏色深淺及格內人數，顯示有空人數／總參與人數。 |
| 最佳時段 | 列出最多人有空的半小時時段，最多先顯示 5 筆。 |
| 鍵盤操作 | 方向鍵移動時間格，Enter 或空白鍵切換選取；Home／End 移到該列首尾。 |

所有日期與時間均以**臺灣時間（UTC+8）**表示。填寫後需要按「儲存我的時間」，才會送出個人選擇。

## 使用流程

1. 主辦人建立活動，取得活動連結及 4 位數代碼。
2. 參與者開啟連結，或在網站首頁輸入代碼加入。
3. 在「我的時間」選取空閒時段，輸入名字並儲存。
4. 切換到「大家的時間」，查看統計與最佳時段。
5. 需要調整時，使用原本的瀏覽器回到活動，再修改並儲存。

若產生代碼失敗，活動仍可能已建立，可先使用活動連結邀請，再依錯誤提示處理代碼設定。

## 專案結構

```text
when-free/
├─ index.html             # 網頁主程式，內含 CSS 與 JavaScript
├─ firebase-rules.json    # Realtime Database 存取規則
├─ README.md
└─ docs/
   ├─ 設定說明與規格.txt
   ├─ 測試紀錄.txt
   └─ *.png              # 開發過程與測試截圖
```

網站前端只有 `index.html` 一個檔案；規則與文件用於雲端設定及專案說明，不需要前端建置流程。

## 本機執行

需要網路連線，以載入 Firebase SDK 並存取雲端資料。

若已安裝 Python，在專案根目錄執行：

```powershell
python -m http.server 8000
```

再用瀏覽器開啟 `http://localhost:8000/`。也可以使用編輯器提供的靜態網頁伺服器。

建議透過 HTTP 本機伺服器或已部署的 HTTPS 網站操作。分享給其他人時，請使用正式網站網址；`localhost` 與 `file:///` 連結不能讓朋友直接開啟你的網站。

## Firebase 設定

目前 `index.html` 已包含此專案的 Firebase 網頁設定。若自行部署另一份作品，請改用自己的 Firebase 專案與資料庫。

### 1. 建立網頁應用程式

在 [Firebase 主控台](https://console.firebase.google.com/) 建立專案並新增網頁應用程式，取得網頁設定。

修改 `index.html` 中的 `firebaseConfig`，確保必要欄位皆已填入：

```js
const firebaseConfig = {
  apiKey: "你的網頁 API Key",
  authDomain: "你的專案.firebaseapp.com",
  databaseURL: "你的 Realtime Database 完整網址",
  projectId: "你的專案 ID",
  appId: "你的網頁 App ID"
};
```

程式已透過 Google CDN 載入 Firebase SDK，不需安裝 npm 套件。

### 2. 啟用匿名登入

進入 **Authentication → 登入方式（Sign-in method）→ 匿名（Anonymous）**，啟用並儲存。網站會自動取得匿名身分，參與者不需註冊帳號。

### 3. 建立 Realtime Database 並發布規則

1. 建立 **Realtime Database**，將其完整網址填入 `databaseURL`。
2. 在資料庫的「規則」頁籤，貼上根目錄 [firebase-rules.json](firebase-rules.json) 的全部內容。
3. 按「發布」。只更新本機規則檔，不會自動更新 Firebase 上的規則。

請使用包含 `events` 與 `eventCodes` 的完整新版規則。若只有原本的活動規則，連結填寫可能仍可使用，但代碼產生或查詢會遭拒絕。

### 資料結構

```text
events/{eventId}/
  title, dates, start, end, owner, createdAt
  joinCode
  participants/{uid}/
    name
    slots/{日期_半小時索引}: true

eventCodes/{四位數代碼}/
  eventId
  owner
```

代碼以交易（transaction）嘗試取得；遇到已占用代碼時會重選，避免直接覆蓋其他活動的代碼。

## GitHub Pages 部署

1. 將最新版 `index.html` 放在 GitHub 儲存庫根目錄並提交。
2. 進入 **Settings → Pages**。
3. Source 選 **Deploy from a branch**，Branch 選 `main`，資料夾選 `/ (root)`，儲存設定。
4. 等待部署完成，使用 Pages 顯示的網址開啟網站。
5. 使用正式網址建立活動，再以另一個瀏覽器或裝置測試加入、填寫及同步。

此儲存庫的網站入口使用 `rainyu.dev/WhenFree/`。若沿用個人網站的網域，專案的 Custom domain 可留白；該欄位只能填網域名稱，不能填包含 `/WhenFree/` 的網址路徑。

修改本機檔案後，必須重新提交並推送到 GitHub，正式網站才會更新。

## 限制與注意事項

- **活動連結與代碼不是密碼。** 知道連結或代碼的人能查看活動、參與者名字及時間，請只分享給活動夥伴。
- 匿名身分保存在瀏覽器中。清除網站資料、更換瀏覽器、使用新的無痕工作階段或更換網域，可能無法修改原本的紀錄。
- 4 位數代碼的空間有限，目前沒有代碼到期、活動刪除或回收介面。
- 最佳時段以半小時為單位統計，目前不會依「會議需要連續幾小時」篩選。
- 目前沒有完整的團隊帳號、成員管理、跨裝置帳號找回或自動儲存功能。
- 網頁依賴 Firebase 與網路；尚未連線時不能建立活動、查詢代碼或送出時間。

## 開發與驗證紀錄

本專案透過與 AI 討論規格、產生程式，再依操作結果反覆調整完成。主要調整包括：

1. 從單檔網頁擴充為 Firebase 多人即時同步。
2. 依參考圖調整為粗框、硬邊陰影與粉彩風格。
3. 參考 UI UX Pro Max 指引，改善時間格大小、鍵盤導覽及非同步儲存處理。
4. 依使用回饋移除多餘提示區與試用模式，將代碼加入區移到建立活動按鈕下方。
5. 將複製成功回饋改為按鈕本身的文字與顏色變化。

依 [測試紀錄](docs/測試紀錄.txt)，已完成兩個獨立瀏覽器來源連線至 Firebase 的實際讀寫與即時更新測試。代碼功能已做程式邏輯測試；紀錄中的雲端代碼測試仍待新版規則發布後確認，手機實機與正式部署後的跨裝置測試亦待補齊。

更多內容可參考 [設定說明與規格](docs/設定說明與規格.txt)。
