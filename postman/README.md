# Postman 測試指南

## 快速開始

### 1. 匯入 Collection
1. 開啟 Postman
2. 點擊 **Import** 按鈕
3. 拖曳 `Google_OAuth_Demo.postman_collection.json` 進去
4. （可選）同時匯入 `Local_Development.postman_environment.json`

### 2. 啟動後端伺服器
```bash
cd /path/to/114-backend
source .venv/bin/activate
uvicorn main:app --reload
```

### 3. 取得 Google ID Token

1. 開啟 [Google OAuth Playground](https://developers.google.com/oauthplayground)

2. 點右上角 **齒輪 ⚙️**，設定：
   - ✅ Use your own OAuth credentials
   - Client ID: `YOUR_GOOGLE_CLIENT_ID`
   - Client Secret: `YOUR_GOOGLE_CLIENT_SECRET`

3. 左側 **Step 1**：
   - Input your own scopes 輸入：`openid email profile`
   - 點擊 **Authorize APIs**
   - 選擇 Google 帳號登入並授權

4. **Step 2**：
   - 點擊 **Exchange authorization code for tokens**
   - 複製回應中的 `id_token`（很長的一串）

### 4. 設定 Postman 變數

方法 A：直接在 Collection 變數設定
1. 點擊 Collection 名稱旁的 `...` → **Edit**
2. 切換到 **Variables** 分頁
3. 在 `google_id_token` 欄位貼上你的 id_token

方法 B：在請求的 Body 中直接修改

### 5. 執行測試

按照順序執行：
1. **0. 測試伺服器連線** - 確認伺服器有啟動
2. **1. Google 登入** - 用 id_token 換 JWT（成功後會自動儲存）
3. **2. 取得當前使用者** - 測試 JWT 驗證

---

## Collection 結構

```
114-Backend Google OAuth Demo/
├── 0. 測試伺服器連線     GET  /
├── 1. Google 登入        POST /auth/google
├── 2. 取得當前使用者      GET  /users/me
└── 3. 查看 API 文件      GET  /docs
```

## 變數說明

| 變數名稱 | 說明 |
|---------|------|
| `base_url` | API 伺服器位址，預設 `http://localhost:8000` |
| `google_id_token` | 從 OAuth Playground 取得的 ID Token |
| `access_token` | 登入成功後自動儲存的 JWT |
| `google_client_id` | Google OAuth Client ID |
| `google_client_secret` | Google OAuth Client Secret |

## 自動化功能

- **登入成功後**：`access_token` 會自動儲存到 Collection 變數
- **受保護路由**：會自動帶入 `Authorization: Bearer {{access_token}}`

## 常見問題

### Q: 出現「無效的 Google Token」？
- id_token 可能已過期（約 1 小時），請重新取得
- 確認 id_token 複製完整，沒有遺漏或多餘空格

### Q: 出現「Not authenticated」？
- 請先執行「1. Google 登入」取得 JWT
- 確認 Authorization header 有正確設定

### Q: 出現「無法驗證憑證」？
- JWT 可能已過期（預設 60 分鐘）
- 請重新登入取得新的 JWT
