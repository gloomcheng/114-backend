# 114-Backend：Google OAuth 教學範例

這是資工系後端課程的教學專案，實作 Google OAuth 登入功能。

## 專案結構

```
114-backend/
├── main.py              # FastAPI 主程式，包含兩種 OAuth 架構
├── google_oauth.py      # Google Token 驗證與交換
├── auth_utils.py        # JWT 簽發與驗證
├── requirements.txt     # Python 依賴套件
├── .env                 # 環境變數（不進版控）
└── postman/             # Postman 測試檔案
```

## 快速開始

```bash
# 1. 建立虛擬環境
uv venv
source .venv/bin/activate

# 2. 安裝依賴
uv pip install -r requirements.txt

# 3. 設定環境變數
cp .env.example .env
# 編輯 .env 填入實際的值

# 4. 啟動伺服器
uvicorn main:app --reload

# 5. 開啟 API 文件
open http://localhost:8000/docs
```

---

# Google OAuth 兩種架構詳解

## 為什麼需要了解這個？

當你在網站上看到「使用 Google 登入」按鈕，背後其實有不同的實作方式。理解這些架構，你才能：
- 選擇適合你專案的方案
- 避免把機密資訊暴露在前端
- 理解為什麼某些程式碼要放後端

---

## 架構 A：Authorization Code Flow

### 流程圖

```
┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐
│  使用者   │      │   前端    │      │   後端   │      │  Google  │
└────┬─────┘      └────┬─────┘      └────┬─────┘      └────┬─────┘
     │                 │                 │                 │
     │ 1. 點擊登入     │                 │                 │
     │────────────────>│                 │                 │
     │                 │                 │                 │
     │      2. 重導向至 Google（帶 client_id）              │
     │<────────────────────────────────────────────────────│
     │                 │                 │                 │
     │ 3. 登入並授權   │                 │                 │
     │─────────────────────────────────────────────────────>
     │                 │                 │                 │
     │      4. 重導向回前端，URL 帶著 code                  │
     │<────────────────────────────────────────────────────│
     │                 │                 │                 │
     │                 │ 5. 把 code      │                 │
     │                 │    傳給後端     │                 │
     │                 │────────────────>│                 │
     │                 │                 │                 │
     │                 │                 │ 6. 用 code +    │
     │                 │                 │    client_secret│
     │                 │                 │    換 tokens    │
     │                 │                 │────────────────>│
     │                 │                 │                 │
     │                 │                 │ 7. 回傳 tokens  │
     │                 │                 │<────────────────│
     │                 │                 │                 │
     │                 │ 8. 驗證後發     │                 │
     │                 │    自家 JWT     │                 │
     │                 │<────────────────│                 │
     │                 │                 │                 │
     │ 9. 登入成功     │                 │                 │
     │<────────────────│                 │                 │
```

### 重點

| 項目 | 說明 |
|------|------|
| **前端做什麼** | 導向 Google → 收到 code → 傳給後端 |
| **後端做什麼** | 用 code + secret 換 token → 驗證 → 發 JWT |
| **client_secret** | 只存在後端，前端永遠不知道 |
| **安全性** | ✅ 較高 |

### 對應程式碼

```python
# main.py
@app.post("/auth/google/code")
async def google_auth_with_code(request: CodeRequest):
    # 後端用 code + secret 換 tokens
    tokens = exchange_code_for_tokens(request.code, request.redirect_uri)
    # ...
```

---

## 架構 B：Google Sign-In SDK Flow

### 流程圖

```
┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐
│  使用者   │      │   前端    │      │   後端   │      │  Google  │
└────┬─────┘      └────┬─────┘      └────┬─────┘      └────┬─────┘
     │                 │                 │                 │
     │ 1. 點擊 Google  │                 │                 │
     │    登入按鈕     │                 │                 │
     │────────────────>│                 │                 │
     │                 │                 │                 │
     │      2. Google SDK 彈出登入視窗                     │
     │<────────────────────────────────────────────────────>
     │                 │                 │                 │
     │ 3. 登入並授權   │                 │                 │
     │─────────────────────────────────────────────────────>
     │                 │                 │                 │
     │                 │ 4. Google 直接  │                 │
     │                 │    給 id_token  │                 │
     │                 │<───────────────────────────────────
     │                 │                 │                 │
     │                 │ 5. 前端把       │                 │
     │                 │    id_token     │                 │
     │                 │    傳給後端     │                 │
     │                 │────────────────>│                 │
     │                 │                 │                 │
     │                 │                 │ 6. 向 Google    │
     │                 │                 │    驗證 token   │
     │                 │                 │────────────────>│
     │                 │                 │                 │
     │                 │                 │ 7. 驗證 OK ✓    │
     │                 │                 │<────────────────│
     │                 │                 │                 │
     │                 │ 8. 發自家 JWT   │                 │
     │                 │<────────────────│                 │
     │                 │                 │                 │
     │ 9. 登入成功     │                 │                 │
     │<────────────────│                 │                 │
```

### 重點

| 項目 | 說明 |
|------|------|
| **前端做什麼** | 用 Google SDK 登入 → 直接拿到 id_token → 傳給後端 |
| **後端做什麼** | 驗證 id_token → 發 JWT |
| **client_secret** | 不需要（沒有「換 token」這步） |
| **安全性** | ⚠️ 一般（但仍安全，因為 id_token 有簽章） |

### 對應程式碼

```python
# main.py
@app.post("/auth/google")
async def google_auth(request: TokenRequest):
    # 直接驗證 id_token（不需要 secret）
    user_info = verify_google_id_token(request.id_token)
    # ...
```

---

## 兩種架構比較

| | 架構 A (Code Flow) | 架構 B (SDK Flow) |
|---|---|---|
| **端點** | `POST /auth/google/code` | `POST /auth/google` |
| **前端傳什麼** | authorization code | id_token |
| **誰換 token** | 後端 | 前端 (Google SDK) |
| **需要 client_secret** | ✅ 後端需要 | ❌ 不需要 |
| **可呼叫 Google API** | ✅ 拿到 access_token | ❌ 只有身份資訊 |
| **實作複雜度** | 較複雜 | 較簡單 |
| **適用情境** | 正式產品、需要 Google API | 快速開發、只需登入 |

---

## 安全性重點

```
┌─────────────────────────────────────────────────────────────┐
│                     🔐 安全性原則                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   client_id     → 公開的，可以放前端                        │
│   client_secret → 機密的，只能放後端                        │
│                                                             │
│   ─────────────────────────────────────────────────────     │
│                                                             │
│   如果 secret 放前端會怎樣？                                │
│                                                             │
│   1. 任何人都能從瀏覽器 DevTools 看到                       │
│   2. 任何人都能假裝是你的應用程式                           │
│   3. 可能被用來發送釣魚攻擊                                │
│   4. Google 可能會停用你的 OAuth 憑證                       │
│                                                             │
│   結論：secret 永遠不能出現在前端程式碼中！                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

# Troubleshooting 常見問題

## Git 相關問題

### 問題：git push 被 GitHub 擋下來（偵測到 secrets）

**錯誤訊息：**
```
remote: error: GH013: Repository rule violations found for refs/heads/main.
remote: - GITHUB PUSH PROTECTION
remote:   Push cannot contain secrets
remote:   —— Google OAuth Client Secret ————————————————
```

**原因：**
GitHub 有內建的 Secret Scanning 功能，會自動偵測程式碼中的 API 金鑰、密碼等機密資訊。

**解法 1：移除 secrets 並重寫歷史（推薦）**

```bash
# 1. 先把檔案中的 secrets 改成佔位符
#    例如：GOCSPX-xxx... → YOUR_GOOGLE_CLIENT_SECRET

# 2. Soft reset 到沒有 secrets 的 commit
git log --oneline                    # 找到乾淨的 commit hash
git reset --soft <clean-commit-hash>

# 3. 重新 commit（這次不包含 secrets）
git add .
git commit -m "feat: implement feature without secrets"

# 4. Force push（因為重寫了歷史）
git push --force origin main
```

**解法 2：允許這個 secret（不推薦，但適合教學用）**

GitHub 錯誤訊息中會提供一個連結，點擊後可以選擇「Allow this secret」。

⚠️ 注意：這樣做會讓 secret 公開在 GitHub 上，僅適合教學用的測試憑證。

---

### 問題：git push 被拒絕（non-fast-forward）

**錯誤訊息：**
```
! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'github.com:xxx/xxx.git'
hint: Updates were rejected because the tip of your current branch is behind
```

**原因：**
遠端有你本地沒有的 commit（可能是別人 push 的，或你在另一台電腦 push 的）。

**解法：**

```bash
# 方法 1：先 pull 再 push（保留兩邊的變更）
git pull --rebase origin main
git push origin main

# 方法 2：強制覆蓋遠端（⚠️ 會丟失遠端的變更）
git push --force origin main
```

---

### 問題：不小心 commit 了不該 commit 的檔案

**情境：** 把 `.env` 或其他機密檔案 commit 了

**解法：**

```bash
# 1. 從 git 追蹤中移除（但保留本地檔案）
git rm --cached .env

# 2. 確認 .gitignore 有加入這個檔案
echo ".env" >> .gitignore

# 3. Commit 這個變更
git add .gitignore
git commit -m "chore: remove .env from tracking"

# 4. 如果已經 push 到遠端，需要清理歷史
#    （參考上面「移除 secrets 並重寫歷史」的步驟）
```

---

### 問題：想要回到之前的版本

```bash
# 查看歷史
git log --oneline

# 方法 1：只是想看之前的程式碼（不修改歷史）
git checkout <commit-hash>
# 看完後回到最新
git checkout main

# 方法 2：想要回復到某個版本，但保留之後的變更記錄
git revert <commit-hash>

# 方法 3：想要完全回到某個版本（⚠️ 會丟失之後的 commit）
git reset --hard <commit-hash>
git push --force origin main
```

---

## Python / FastAPI 相關問題

### 問題：ModuleNotFoundError: No module named 'xxx'

**解法：**

```bash
# 1. 確認虛擬環境已啟動
source .venv/bin/activate

# 2. 安裝依賴
uv pip install -r requirements.txt

# 3. 如果還是不行，重建虛擬環境
rm -rf .venv
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt
```

---

### 問題：環境變數讀不到

**症狀：** `GOOGLE_CLIENT_ID` 是 `None`

**解法：**

```bash
# 1. 確認 .env 檔案存在
ls -la .env

# 2. 確認 .env 內容正確（沒有多餘的空格或引號）
cat .env

# 正確格式：
GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com

# 錯誤格式：
GOOGLE_CLIENT_ID = "xxx"   # 不要有空格和引號
```

---

### 問題：「無效的 Google Token」

**可能原因：**

1. **Token 過期** - id_token 約 1 小時後過期，重新取得即可
2. **Client ID 不符** - 確認 `.env` 中的 `GOOGLE_CLIENT_ID` 與 OAuth Playground 設定的一致
3. **Token 複製不完整** - id_token 很長，確認完整複製

---

## OAuth Playground 相關問題

### 問題：Authorization Error - redirect_uri_mismatch

**原因：**
OAuth Playground 的 redirect URI 沒有在 Google Cloud Console 中設定。

**解法：**

1. 到 [Google Cloud Console](https://console.cloud.google.com/apis/credentials)
2. 點選你的 OAuth 2.0 Client ID
3. 在「Authorized redirect URIs」加入：
   ```
   https://developers.google.com/oauthplayground
   ```

---

# AI 時代的學習方法

## 當你看不懂 AI 生成的答案時

在 AI 時代，我們經常遇到這種情況：

> 「AI 給了我一段程式碼，但我完全看不懂它在幹嘛...」

這很正常！以下是有效的互動策略：

### 策略 1：請 AI 解釋流程，而不是直接給程式碼

```
❌ 不好的問法：
「幫我實作 Google OAuth 登入」

✅ 好的問法：
「請用流程圖解釋 Google OAuth 登入的運作原理，
不要給我程式碼，我想先理解概念」
```

### 策略 2：追問「為什麼」

```
❌ 只問 What：
「這段程式碼是什麼意思？」

✅ 追問 Why：
「為什麼 client_secret 不能放前端？」
「為什麼需要兩次 redirect？」
「為什麼要驗證 id_token？不驗證會怎樣？」
```

### 策略 3：請 AI 比較不同方案

```
「OAuth 有哪些不同的 flow？各自適合什麼情境？」
「Authorization Code Flow 和 Implicit Flow 差在哪？」
「為什麼現在不推薦用 Implicit Flow？」
```

### 策略 4：請 AI 畫圖或用類比

```
「可以用生活中的例子解釋 OAuth 嗎？」
「請畫一個時序圖說明登入流程」
「id_token 和 access_token 的差別，可以用比喻解釋嗎？」
```

### 策略 5：指出你的困惑點

```
❌ 太籠統：
「我看不懂」

✅ 具體說明困惑：
「我看到流程圖中有一步是 302 redirect，
但我不懂為什麼 Google 不直接回傳資料，
而是要 redirect 回我的網站？」
```

### 策略 6：請 AI 扮演老師

```
「假設我完全不懂 OAuth，
請用蘇格拉底式問答法，
一步步引導我理解這個概念」
```

---

## 實際案例：這個專案的誕生過程

這個教學專案就是透過與 AI 互動完成的，以下是實際的對話摘要：

### 第一輪：請 AI 生成程式碼
```
我：「請幫我實作 Google OAuth」
AI：（生成了程式碼）
```

### 第二輪：發現問題，追問
```
我：「你的流程圖顯示前端在換 token，
     但我上課跟學生說這應該在後端做，
     可以解釋一下嗎？」

AI：「你說得對！讓我重新說明...
     (詳細解釋了兩種架構的差異)」
```

### 第三輪：請 AI 補充另一種架構
```
我：「請先 commit 目前的版本，
     再補上 Authorization Code Flow，
     這樣學生可以比較兩種架構」

AI：（完成兩個 commit，清楚區分兩種實作）
```

### 學到的經驗

1. **AI 不會主動告訴你它的假設**
   - 它假設你想要最簡單的方案（架構 B）
   - 但最簡單不一定最適合教學

2. **你的領域知識很重要**
   - 因為我知道「token 交換應該在後端」
   - 才能發現 AI 的流程圖有誤導

3. **對話式學習比單次提問更有效**
   - 透過追問和質疑，得到更深入的理解
   - AI 的「錯誤」反而是很好的教學素材

---

## 給學生的建議

```
┌─────────────────────────────────────────────────────────────┐
│                  🎓 AI 時代的學習心法                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. 不要只複製貼上 AI 的答案                                │
│     → 問它「為什麼這樣寫」                                  │
│                                                             │
│  2. 發現看不懂的地方，是學習的起點                          │
│     → 把困惑告訴 AI，請它解釋                               │
│                                                             │
│  3. 嘗試質疑 AI 的答案                                      │
│     → AI 不一定對，你的懷疑可能是對的                       │
│                                                             │
│  4. 請 AI 用不同方式解釋                                    │
│     → 圖解、類比、對比、舉例                                │
│                                                             │
│  5. 先理解概念，再看程式碼                                  │
│     → 「先不要給程式碼，解釋原理就好」                      │
│                                                             │
│  6. 建立自己的知識框架                                      │
│     → AI 是工具，但理解是你自己的                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 測試方式

請參考 [postman/README.md](./postman/README.md) 的說明。

---

## License

MIT - 教學用途，歡迎自由使用
