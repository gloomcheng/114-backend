# Render 部署教學 / Render Deployment Guide

本文件手把手帶你把 FastAPI 應用程式部署到 Render 雲端平台。
This guide walks you through deploying a FastAPI application to the Render cloud platform.

---

## 前置準備 / Prerequisites

在開始之前，請確認：
Before starting, please confirm:

- [x] 程式碼已經 push 到 GitHub（public 或 private 都可以）
      Code has been pushed to GitHub (public or private)
- [x] 專案根目錄有 `Dockerfile`
      Project root has a `Dockerfile`
- [x] 本地端 `docker build` 和 `docker run` 都能成功
      Local `docker build` and `docker run` both succeed

---

## Step 1：建立 Render 帳號 / Create Render Account

1. 前往 / Go to [render.com](https://render.com)
2. 點擊右上角 **Get Started** / Click **Get Started** in the upper right
3. **建議使用 GitHub 帳號登入** / **Recommended: Log in with GitHub account**
   （之後連結 repo 更方便 / Makes connecting repos easier later）

---

## Step 2：建立 Web Service / Create Web Service

1. 登入後，點擊右上角 **New** 按鈕
   After logging in, click the **New** button in the upper right
2. 選擇 / Select **Web Service**

```
┌─────────────────────────────────────────┐
│  New ▼                                  │
├─────────────────────────────────────────┤
│  ● Web Service        ← 選這個 Select   │
│  ○ Static Site                          │
│  ○ Background Worker                    │
│  ○ Cron Job                             │
│  ○ Private Service                      │
│  ○ PostgreSQL                           │
│  ○ Redis                                │
└─────────────────────────────────────────┘
```

3. 選擇 / Select **Build and deploy from a Git repository**
4. 點擊 / Click **Connect GitHub** or **Connect GitLab**

---

## Step 3：選擇 Repository / Select Repository

1. 如果是第一次使用，需要授權 Render 存取你的 GitHub
   First time users need to authorize Render to access GitHub
2. 點擊 / Click **Configure account** → 選擇要授權的 repositories / Select repositories to authorize
3. 找到你的 `114-backend` repository，點擊 / Find and click **Connect**

---

## Step 4：設定服務 / Configure Service

填寫以下欄位 / Fill in these fields:

| 欄位 Field | 說明 Description | 建議值 Recommended |
|------|------|--------|
| **Name** | 服務名稱，會變成網址的一部分 / Service name, becomes part of URL | `114-backend` |
| **Region** | 伺服器位置 / Server location | `Singapore (Southeast Asia)` |
| **Branch** | 要部署哪個分支 / Branch to deploy | `main` |
| **Root Directory** | 專案根目錄 / Project root | _(留空 leave empty)_ |
| **Runtime** | 如何執行程式 / How to run | `Docker` |

### Runtime 設定 / Runtime Selection

```
┌─────────────────────────────────────────┐
│  Runtime                                │
├─────────────────────────────────────────┤
│  ○ Node                                 │
│  ○ Python                               │
│  ○ Go                                   │
│  ○ Rust                                 │
│  ● Docker        ← 選這個 Select this   │
└─────────────────────────────────────────┘
```

> 💡 因為我們有 `Dockerfile`，選 Docker 可以讓 Render 用我們定義好的環境。
>    Since we have a `Dockerfile`, selecting Docker lets Render use our defined environment.

### 方案選擇 Instance Type / Pricing Plans

| 方案 Plan | 價格 Price | 規格 Specs | 適合 Best For |
|------|------|------|------|
| **Free** | $0 | 512 MB RAM | 教學 Learning / Demo |
| Starter | $7/月 | 512 MB RAM | Side Project |
| Standard | $25/月 | 2 GB RAM | 小型產品 Small products |

教學用選 **Free** 即可。
For learning, **Free** is sufficient.

> ⚠️ **Free 方案限制 / Free Plan Limitation**：
> 15 分鐘無流量會自動休眠，下次請求需等待喚醒（約 30 秒）
> Auto-sleeps after 15 minutes of inactivity, next request waits ~30s to wake up

---

## Step 5：設定環境變數 / Set Environment Variables

往下滑到 **Environment Variables** 區塊，點擊 **Add Environment Variable**：
Scroll down to **Environment Variables** section, click **Add Environment Variable**:

| Key | Value | 說明 Description |
|-----|-------|------|
| `GOOGLE_CLIENT_ID` | `xxx.apps.googleusercontent.com` | 從 Google Cloud Console 取得 / From Google Cloud Console |
| `GOOGLE_CLIENT_SECRET` | `GOCSPX-xxx` | 從 Google Cloud Console 取得 / From Google Cloud Console |
| `JWT_SECRET_KEY` | `your-super-secret-key` | 自己設一個複雜的字串 / Set your own complex string |

```
┌─────────────────────────────────────────────────────────────┐
│  Environment Variables                                       │
├─────────────────────────────────────────────────────────────┤
│  GOOGLE_CLIENT_ID      = xxx.apps.googleusercontent.com     │
│  GOOGLE_CLIENT_SECRET  = ●●●●●●●●●● (Secret)               │
│  JWT_SECRET_KEY        = ●●●●●●●●●● (Secret)               │
│                                                             │
│  [+ Add Environment Variable]                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Step 6：部署！ / Deploy!

1. 確認所有設定都正確 / Confirm all settings are correct
2. 點擊最下方的 **Create Web Service** / Click **Create Web Service** at the bottom
3. Render 會開始 / Render will start:
   - Clone 你的 repo / Clone your repo
   - 執行 `docker build` / Run `docker build`
   - 啟動 container / Start container

```
Build Logs:
──────────────────────────────────────────
Cloning repo...
Building docker image...
Step 1/7 : FROM python:3.12-slim
Step 2/7 : WORKDIR /app
...
✅ Deploy successful!
──────────────────────────────────────────
```

等待約 2-3 分鐘，直到看到 **Live** 狀態。
Wait about 2-3 minutes until you see **Live** status.

---

## Step 7：測試你的 API / Test Your API

部署成功後，你會得到一個網址 / After successful deployment, you'll get a URL:

```
https://你的服務名稱.onrender.com
https://your-service-name.onrender.com
```

測試方式 / Testing:

```bash
# 測試首頁 / Test homepage
curl https://114-backend.onrender.com/

# 應該回傳 / Should return:
# {"message":"Hello FastAPI OAuth Demo"}
```

或直接在瀏覽器打開 / Or open in browser:

```
https://114-backend.onrender.com/docs
```

會看到 FastAPI 的 Swagger UI 文件頁面！
You'll see the FastAPI Swagger UI documentation page!

---

## Step 8：設定 Deploy Hook（選用）/ Set up Deploy Hook (Optional)

如果你想讓 GitHub Actions 自動觸發 Render 部署，需要設定 Deploy Hook。
To let GitHub Actions automatically trigger Render deployment, set up a Deploy Hook.

### 8.1 取得 Deploy Hook URL / Get Deploy Hook URL

1. 在 Render Dashboard，點進你的服務
   In Render Dashboard, click on your service
2. 左側選單點擊 **Settings** / Click **Settings** in left menu
3. 往下滑到 **Deploy Hook** 區塊 / Scroll down to **Deploy Hook** section
4. 點擊 **Generate Deploy Hook** / Click **Generate Deploy Hook**
5. 複製產生的 URL / Copy the generated URL
   （長得像 `https://api.render.com/deploy/srv-xxx?key=xxx`）

### 8.2 設定 GitHub Secret / Configure GitHub Secret

1. 到你的 GitHub Repository / Go to your GitHub Repository
2. 點擊 / Click **Settings** → **Secrets and variables** → **Actions**
3. 點擊 / Click **New repository secret**
4. 填寫 / Fill in:
   - **Name**: `RENDER_DEPLOY_HOOK_URL`
   - **Value**: 貼上剛才複製的 URL / Paste the URL you copied
5. 點擊 / Click **Add secret**

現在每次 push 到 main 分支，GitHub Actions 就會自動觸發 Render 重新部署！
Now every push to main branch will automatically trigger Render redeployment via GitHub Actions!

---

## 常見問題 / FAQ

### Q: 部署失敗，顯示 "Build failed" / Deploy failed with "Build failed"

**可能原因 / Possible causes:**
1. `Dockerfile` 有語法錯誤 / Dockerfile has syntax errors
2. `requirements.txt` 有套件安裝失敗 / Package installation failed

**解法 / Solution:**
先在本地測試 `docker build`，確認能成功再 push。
Test `docker build` locally first, confirm it succeeds before pushing.

---

### Q: 部署成功，但 API 回傳 500 Error / Deploy succeeded but API returns 500 Error

**可能原因 / Possible causes:**
1. 環境變數沒設定或設定錯誤 / Environment variables not set or incorrect
2. 程式碼有 bug / Code has bugs

**解法 / Solution:**
1. 檢查 Render Dashboard 的 Logs / Check Logs in Render Dashboard
2. 確認所有環境變數都有設定 / Confirm all environment variables are set

---

### Q: Free 方案服務很慢？/ Free plan service is slow?

**原因 / Reason:**
Free 方案會在 15 分鐘無流量後休眠，喚醒需要 30 秒左右。
Free plan sleeps after 15 minutes of inactivity, waking up takes ~30 seconds.

**解法 / Solution:**
- 教學用可以接受 / Acceptable for learning
- 正式產品請升級付費方案 / Upgrade to paid plan for production

---

### Q: 如何更新 Google Cloud Console 的 Redirect URI? / How to update Redirect URI in Google Cloud Console?

部署到 Render 後，需要在 Google Cloud Console 加入新的 redirect URI：
After deploying to Render, add new redirect URI in Google Cloud Console:

1. 到 / Go to [Google Cloud Console](https://console.cloud.google.com/apis/credentials)
2. 點擊你的 OAuth 2.0 Client ID / Click your OAuth 2.0 Client ID
3. 在 **Authorized redirect URIs** 加入 / Add to **Authorized redirect URIs**:
   ```
   https://你的服務名稱.onrender.com/auth/google/callback
   https://your-service-name.onrender.com/auth/google/callback
   ```
4. 點擊 / Click **Save**

---

## 小結 / Summary

恭喜！你已經成功把 FastAPI 部署到雲端！
Congratulations! You've successfully deployed FastAPI to the cloud!

```
┌─────────────────────────────────────────────────────────────┐
│              🎉 部署完成！ Deployment Complete!              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   你的 API 網址 Your API URL：                              │
│   https://你的服務名稱.onrender.com                          │
│   https://your-service-name.onrender.com                    │
│                                                             │
│   API 文件 Documentation：                                  │
│   https://你的服務名稱.onrender.com/docs                     │
│                                                             │
│   完整流程 Full Flow：                                      │
│   git push → GitHub Actions 測試 test                       │
│           → 自動部署 auto deploy → Render                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

有任何問題，請詢問老師！
If you have any questions, please ask your instructor!
