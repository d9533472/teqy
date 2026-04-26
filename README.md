# 淨慈寺文化園區 CMS

完整的迷你 CMS 系統,用來管理 https://teqy.weebly.com 改版後的網站。

## 架構

```
GitHub Pages (靜態託管)
├── index.html          ← 公開網站
└── admin.html          ← 後台 CMS (密碼登入)
                         ↓ XHR
                Google Apps Script
                         ↓
              Google Drive
              ├── content.json   ← 所有頁面/區塊資料
              ├── backups/       ← 自動備份(保留最近 30 版)
              └── (媒體資料夾)/   ← 上傳的圖片
```

## 部署步驟

### 1. 建立 Google Drive 兩個資料夾

- 資料夾 A:用來放 `content.json` 和備份(例:`JingCi-Content`)
- 資料夾 B:用來放上傳的圖片(例:`JingCi-Media`)

各自打開資料夾,從網址列複製資料夾 ID(網址 `/folders/` 後面那串)。

### 2. 建立 Google Apps Script 專案

1. 開啟 [https://script.google.com](https://script.google.com),新增專案
2. 把 `code.gs` 內容貼進去
3. 修改最上方 `CONFIG`:
   ```javascript
   const CONFIG = {
     PASSWORD: '改成你要的密碼',
     CONTENT_FOLDER_ID: '貼上資料夾 A 的 ID',
     MEDIA_FOLDER_ID: '貼上資料夾 B 的 ID',
     // 其他保持預設
   };
   ```
4. **編輯器執行 `initContent()` 一次** — 觸發 Drive 授權(會跳權限視窗,要全部允許)。看到 log「Content folder OK」「Media folder OK」就 OK。
5. 部署 → 新增部署作業 → 類型選「網頁應用程式」
   - 執行身分:**我**
   - 存取權:**任何人**
6. 部署後複製「網頁應用程式 URL」(以 `https://script.google.com/macros/s/.../exec` 結尾)

### 3. 把 GAS URL 貼進兩個 HTML 檔

打開 `index.html` 和 `admin.html`,各找這一行:

```javascript
const GAS_URL = 'YOUR_GAS_WEB_APP_URL_HERE';
```

把單引號裡面換成你剛複製的 GAS URL。

### 4. 推到 GitHub + 開 Pages

1. 把 `index.html` 和 `admin.html` 推到一個 GitHub repo(可用既有的 `d9533472` repo,新建子資料夾)
2. Settings → Pages → Source 設定為對應 branch + 路徑
3. 等幾分鐘,Pages 連結啟用後:
   - 公開頁:`https://你的帳號.github.io/repo名稱/`
   - 後台:`https://你的帳號.github.io/repo名稱/admin.html`

### 5. 第一次使用

1. 打開 `admin.html` 連結 → 輸入剛設的密碼登入
2. 第一次登入時 GAS 會自動把預設內容(從原 Weebly 抓的資料)寫成 `content.json`
3. 開始編輯,點右上角「儲存所有變更」即可

## 功能總覽

### 後台支援的區塊類型(可任意組合到任何頁面)

| 類型 | 用途 |
|------|------|
| Hero | 首頁主視覺,有大標、副標、背景圖、CTA 按鈕 |
| 角框卡片 | 左側標題區 + 右側引文 + 卡片網格(原「興建因緣」風格) |
| 時間軸 | 直立時間軸(原「興建進度」風格) |
| 漸層卡片 | 4 欄漸層卡片(原「空間規劃」風格) |
| 年份檔案 | 6 欄年份卡 + 附加 YouTube 格(原「影音中心」風格) |
| 捐款卡片 | 雙欄捐款卡 + 額外連結(原「捐款方式」風格) |
| 富文字段落 | Quill 編輯器,支援標題、粗斜體、清單、連結、顏色 |
| 單張圖片 | 圖片 + 圖說 |
| YouTube 影片 | 嵌入 YouTube |
| 分隔線 | 裝飾性分隔 |

### 頁面管理
- 新增/刪除/重新命名頁面
- 設定 URL slug(網址會是 `#/page/your-slug`)
- 標記「首頁」(只能有一個)

### 導覽列
- 新增/刪除/排序選項
- 三種類型:錨點(`#section`)、頁面(內部頁)、外部連結
- 支援一層下拉子選單

### 媒體
- 上傳圖片 → 自動進 Drive 的 Media 資料夾 → 取得 `lh3.googleusercontent.com` 直連
- 也可以直接貼網址用外部圖

### 富文字編輯器(Quill)
工具列:標題級別、粗體、斜體、底線、刪除線、清單、顏色、連結、引用區塊

### 自動備份
每次儲存前都會把舊的 `content.json` 移到 `backups/` 子資料夾並加時間戳。後台「備份歷史」可以看清單並一鍵還原。保留最近 30 個版本,舊的會自動刪除。

### 公開頁快取
GAS 端用 CacheService 快取 10 分鐘,避免每個訪客都打資料庫。儲存時會自動清快取。

## 進階設定

### 加快公開頁載入
首次載入會打 GAS,大約 0.5–1.5 秒。如果想再更快可以:
- 把 `content.json` 設為公開,直接從 Drive 抓(免 GAS quota)
- 或把 JSON 嵌進 `index.html` 當 fallback,GAS 載入後再覆蓋

### 修改密碼
直接改 `code.gs` 裡的 `CONFIG.PASSWORD`,然後重新部署一次(部署 → 管理部署作業 → 編輯 → 儲存)。

### 新增區塊類型(進階)
1. 在 `admin.html` 的 `SECTION_SCHEMAS` 加一個新 type,定義 fields
2. 在 `index.html` 的 `SECTION_RENDERERS` 加對應的 React 元件

## Troubleshooting

| 症狀 | 解法 |
|------|------|
| 後台一直「載入中⋯」 | 檢查 `GAS_URL` 是否正確、GAS 部署存取權是否設「任何人」 |
| 圖片上傳失敗:`Drive permission` | 在 GAS 編輯器再執行一次 `initContent()`,然後**重新部署**(很重要) |
| 圖片上傳成功但顯示不出來 | Drive 檔案分享權限沒設好,GAS 程式裡的 `setSharing` 應該已經處理。如果還是壞掉,可以手動到 Drive 把 Media 資料夾設為「知道連結的任何人都可以檢視」 |
| 修改密碼後後台跳不回登入畫面 | 在後台網址後加 `?logout` 或開無痕視窗 |
| 公開頁看不到剛改的內容 | GAS 快取了 10 分鐘。重新儲存一次會自動清,或者等 10 分鐘 |

## 檔案清單

- `code.gs` — Google Apps Script 後端(部署成 Web App)
- `index.html` — 公開網站(部署到 GitHub Pages)
- `admin.html` — 後台 CMS(部署到 GitHub Pages,密碼登入)
- `README.md` — 本說明文件
