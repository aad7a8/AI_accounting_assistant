# API 設計規範 (API Design Specification) - CoffeeBook AI API

---

**文件版本 (Document Version):** `v1.1.0`
**最後更新 (Last Updated):** `2026-02-11`
**主要作者/設計師 (Lead Author/Designer):** `[Backend Engineer]`
**審核者 (Reviewers):** `[Lead Engineer, PM, Frontend Developer]`
**狀態 (Status):** `草稿 (Draft)`
**相關架構文檔:** `05_architecture_design.md`
**對應 PRD:** `02_prd.md v1.5`
**專案代號:** `CB-2026-Q1`
**OpenAPI 定義文件:** `TBD — 將於開發階段由程式碼自動產生`

**相關文件：**
- 系統架構與技術選型 → [架構設計文件 (05_architecture_design.md)](05_architecture_design.md)
- 使用者故事與驗收標準 → [PRD (02_prd.md)](02_prd.md)
- BDD 行為場景 → [BDD 情境文件 (03_bdd_scenarios.md)](03_bdd_scenarios.md)
- 模組層級契約與測試 → [模組規格與測試 (07_module_specification_and_tests.md)](07_module_specification_and_tests.md)

---

## 目錄 (Table of Contents)

1. [引言 (Introduction)](#1-引言-introduction)
2. [設計原則與約定 (Design Principles and Conventions)](#2-設計原則與約定-design-principles-and-conventions)
3. [認證與授權 (Authentication and Authorization)](#3-認證與授權-authentication-and-authorization)
4. [通用 API 行為 (Common API Behaviors)](#4-通用-api-行為-common-api-behaviors)
   - [4.5 離線同步 (Offline Sync)](#45-離線同步-offline-sync)
5. [錯誤處理 (Error Handling)](#5-錯誤處理-error-handling)
6. [安全性考量 (Security Considerations)](#6-安全性考量-security-considerations)
7. [API 端點詳述 (API Endpoint Definitions)](#7-api-端點詳述-api-endpoint-definitions)
   - [7.1 資源：認證 (Auth)](#71-資源認證-auth)
   - [7.2 資源：使用者 (Users)](#72-資源使用者-users)
   - [7.3 資源：交易記錄 (Transactions)](#73-資源交易記錄-transactions)
   - [7.4 資源：語音輸入 (Voice)](#74-資源語音輸入-voice)
   - [7.5 資源：快速按鈕 (Quick Buttons)](#75-資源快速按鈕-quick-buttons)
   - [7.6 資源：儀表板 (Dashboard)](#76-資源儀表板-dashboard)
   - [7.7 資源：月報表 (Reports)](#77-資源月報表-reports)
   - [7.8 資源：系統健康 (Health)](#78-資源系統健康-health)
   - [7.9 資源：離線同步 (Sync)](#79-資源離線同步-sync)
8. [資料模型/Schema 定義 (Data Models / Schema Definitions)](#8-資料模型schema-定義-data-models--schema-definitions)
9. [API 生命週期與版本控制 (API Lifecycle and Versioning)](#9-api-生命週期與版本控制-api-lifecycle-and-versioning)
10. [附錄 (Appendix)](#10-附錄-appendix)

---

## 1. 引言 (Introduction)

### 1.1 目的 (Purpose)

為「家庭咖啡店智能記帳系統 (CoffeeBook AI)」的前端消費者 (React Native App / React Web) 與後端實現者提供一個統一、明確、易於遵循的 API 接口契約。本系統為本地部署架構，API Server 運行在店內設備上。

### 1.2 目標讀者 (Target Audience)

- 前端開發者 (React Native App / React Web SPA)
- 後端開發者 (Node.js API Server)
- QA 測試工程師

### 1.3 快速入門 (Quick Start)

**第 1 步：登入取得 Token**

```bash
curl --request POST \
  --url https://192.168.1.100/api/v1/auth/login \
  --header 'Content-Type: application/json' \
  --data '{
    "email": "owner@coffeeshop.local",
    "password": "your_password"
  }'
```

**預期回應 (200 OK)：**

```json
{
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "id": "usr_01HXYZ...",
      "name": "店主",
      "email": "owner@coffeeshop.local",
      "role": "owner"
    }
  }
}
```

**第 2 步：使用 Token 呼叫 API**

```bash
curl --request GET \
  --url https://192.168.1.100/api/v1/dashboard/today \
  --header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIs...'
```

**第 3 步：驗證系統健康**

```bash
curl --request GET \
  --url https://192.168.1.100/api/v1/health
```

**預期回應：**

```json
{
  "status": "ok",
  "version": "1.0.0",
  "database": "connected",
  "uptime_seconds": 86400
}
```

---

## 2. 設計原則與約定 (Design Principles and Conventions)

### 2.1 API 風格 (API Style)

- **風格：** RESTful
- **核心原則：** 資源導向、無狀態、標準 HTTP 方法
- **設計哲學：** 簡潔為上。本系統為單店專屬部署，API 消費者僅為自家前端 App，因此省略 HATEOAS、OAuth Scopes 等面向公開 API 的複雜機制

### 2.2 基本 URL (Base URL)

- **本地網路 (Production)：** `https://{LOCAL_SERVER_IP}/api/v1` (Nginx Reverse Proxy + TLS 1.3)
- **開發環境 (Development)：** `http://localhost:3000/api/v1`
- **Tailscale VPN (完整版選配)：** `https://coffeebook.tailnet-xxxx.ts.net/api/v1`

### 2.3 請求與回應格式 (Request and Response Formats)

- **格式：** `application/json` (UTF-8 編碼)
- **檔案上傳：** `multipart/form-data` (照片附件、快速按鈕圖片)
- **語音上傳：** `multipart/form-data` (音檔 .wav / .m4a)

### 2.4 標準 HTTP Headers

**所有請求：**

| Header | 必填 | 說明 |
| :--- | :--- | :--- |
| `Authorization` | 是 (除 `/auth/login` 和 `/health`) | `Bearer <JWT_TOKEN>` |
| `Content-Type` | 是 | `application/json` 或 `multipart/form-data` |
| `X-Request-ID` | 否 | UUID，用於追蹤請求；未提供時伺服器自動生成 |

**所有回應：**

| Header | 說明 |
| :--- | :--- |
| `X-Request-ID` | 回傳請求追蹤 ID |
| `X-RateLimit-Limit` | 速率上限 |
| `X-RateLimit-Remaining` | 剩餘配額 |

### 2.5 命名約定 (Naming Conventions)

- **資源路徑：** 小寫，連字符分隔，名詞複數 (e.g., `/transactions`, `/quick-buttons`)
- **JSON 欄位：** `snake_case` (e.g., `item_name`, `created_at`, `total_revenue`)
- **查詢參數：** `snake_case` (e.g., `page_size`, `sort_by`, `start_date`)

### 2.6 日期與時間格式 (Date and Time Formats)

- **標準格式：** ISO 8601 + UTC (e.g., `2026-02-11T08:30:00Z`)
- **僅日期：** `YYYY-MM-DD` (e.g., `2026-02-11`)
- **月份 period：** `YYYY-MM` (e.g., `2026-02`)

---

## 3. 認證與授權 (Authentication and Authorization)

### 3.1 認證機制 (Authentication Mechanism)

- **機制：** JWT (JSON Web Token)，本地簽發
- **Token 傳遞：** `Authorization: Bearer <token>`
- **Token 有效期：** 24 小時
- **Token 簽發：** 僅透過 `POST /api/v1/auth/login`
- **無 Refresh Token：** 本地部署場景下，Token 過期後重新登入即可 (MVP 簡化)

**JWT Payload 結構：**

```json
{
  "user_id": "usr_01HXYZ...",
  "role": "owner",
  "iat": 1708675200,
  "exp": 1708761600
}
```

### 3.2 授權模型 (Authorization Model)

- **模型：** 基於角色的存取控制 (RBAC)
- **角色定義：**

| 角色 | 說明 | 記帳 | 查看報表 | 管理帳號 | 管理設定 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `owner` | 咖啡店店主 | ✅ | ✅ | ✅ | ✅ |
| `manager` | 管理者員工 | ✅ | ✅ | ❌ | ❌ |
| `employee` | 一般員工 | ✅ | ❌ | ❌ | ❌ |

- **權限失敗回應：** `403 Forbidden`
- **認證失敗回應：** `401 Unauthorized`

---

## 4. 通用 API 行為 (Common API Behaviors)

### 4.1 分頁 (Pagination)

- **策略：** Offset-based 分頁 (單店資料量 < 50K 筆，不需 Cursor-based)
- **查詢參數：**
  - `page` — 頁碼，預設 `1`
  - `page_size` — 每頁筆數，預設 `20`，最大 `100`
- **回應結構：**

```json
{
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total_count": 156,
    "total_pages": 8
  }
}
```

### 4.2 排序 (Sorting)

- **查詢參數：** `sort_by`
- **格式：** `sort_by=created_at` (升序), `sort_by=-created_at` (降序，預設)
- **可排序欄位：** 各端點中個別定義

### 4.3 過濾 (Filtering)

- **策略：** 直接使用欄位名作為查詢參數
- **日期範圍：** `start_date=2026-02-01&end_date=2026-02-28`
- **列舉過濾：** `type=income`, `category=進貨成本`
- **記錄者過濾：** `user_id=usr_01HXYZ...`
- **範例：** `GET /api/v1/transactions?type=expense&start_date=2026-02-01&sort_by=-amount`

### 4.4 標準成功回應封裝 (Standard Success Response)

所有成功回應均使用統一封裝：

**單一資源：**

```json
{
  "data": { ... }
}
```

**列表資源：**

```json
{
  "data": [ ... ],
  "pagination": { ... }
}
```

**無內容 (204 No Content)：** 回應體為空。

### 4.5 離線同步 (Offline Sync)

> 對齊 PRD NFR：「可離線記帳，網路恢復後自動同步」

- **策略：** Client-Generated ID + Batch Push + Incremental Pull
- **核心概念：**
  1. 前端在離線時將交易儲存於本地 (SQLite / AsyncStorage)，每筆記錄帶有 `client_id` (UUID v4)
  2. 網路恢復後，前端呼叫 `POST /sync/push` 批量上傳離線資料
  3. 前端呼叫 `GET /sync/pull` 拉取其他裝置 / 使用者產生的新變更
  4. 伺服器以 `client_id` 做冪等去重，防止重複建立
- **衝突解決：** Server-wins — 若同一筆記錄在離線期間被多端修改，以伺服器時間戳較新者為準；被覆寫的變更在 `push` 回應中標記 `conflict`，由前端提示使用者
- **同步範圍：** `transactions` 與 `quick_buttons` 兩類資源
- **同步頻率：** 網路恢復後自動觸發一次；之後每 5 分鐘輪詢 (可由前端設定調整)

---

## 5. 錯誤處理 (Error Handling)

### 5.1 標準錯誤回應格式

```json
{
  "error": {
    "type": "invalid_request_error",
    "code": "parameter_missing",
    "message": "金額為必填欄位",
    "param": "amount",
    "request_id": "req_abc123"
  }
}
```

### 5.2 通用 HTTP 狀態碼

| 狀態碼 | 說明 | 使用場景 |
| :--- | :--- | :--- |
| `200 OK` | 成功 | GET / PATCH 成功 |
| `201 Created` | 資源已建立 | POST 成功建立 |
| `204 No Content` | 成功，無回應體 | DELETE 成功 |
| `400 Bad Request` | 請求參數錯誤 | 驗證失敗 |
| `401 Unauthorized` | 認證失敗 | Token 無效/過期 |
| `403 Forbidden` | 權限不足 | 角色無權存取 |
| `404 Not Found` | 資源不存在 | ID 不存在 |
| `409 Conflict` | 資源衝突 | Email 重複 |
| `422 Unprocessable Entity` | 語義錯誤 | 業務規則違反 |
| `429 Too Many Requests` | 超出速率限制 | 登入嘗試過頻 |
| `500 Internal Server Error` | 伺服器內部錯誤 | 未預期例外 |

### 5.3 錯誤碼字典 (Error Code Dictionary)

| `error.code` | HTTP 狀態碼 | 描述 |
| :--- | :--- | :--- |
| `authentication_failed` | 401 | 帳號或密碼錯誤 |
| `token_expired` | 401 | JWT Token 已過期 |
| `token_invalid` | 401 | JWT Token 格式無效 |
| `permission_denied` | 403 | 角色無權執行此操作 |
| `resource_not_found` | 404 | 請求的資源不存在 |
| `parameter_missing` | 400 | 缺少必填參數 |
| `parameter_invalid` | 400 | 參數格式或值不合法 |
| `email_already_exists` | 409 | 此 Email 已被使用 |
| `account_limit_exceeded` | 422 | 帳號數量已達上限 (10 個) |
| `quantity_invalid` | 422 | 數量必須大於 0 |
| `voice_recognition_failed` | 422 | 語音辨識失敗，無法解析 |
| `classification_failed` | 422 | AI 分類失敗 (含 Fallback 皆失敗) |
| `report_not_ready` | 422 | 該月報表尚未產生 |
| `network_required` | 503 | 此功能需要網路連線 (語音辨識) |
| `rate_limit_exceeded` | 429 | 超出速率限制 |
| `sync_conflict` | 409 | 離線同步衝突，伺服器版本較新 |
| `sync_batch_too_large` | 400 | 同步批次超過 100 筆上限 |
| `internal_server_error` | 500 | 伺服器內部錯誤 |

---

## 6. 安全性考量 (Security Considerations)

### 6.1 傳輸層安全 (TLS)

- **所有 Production API 請求強制使用 HTTPS + TLS 1.3** (對齊 PRD NFR 要求)
- 本地網路：Nginx Reverse Proxy 終止 TLS (使用 `mkcert` 自簽憑證或 Let's Encrypt)
- Tailscale VPN (完整版)：端對端加密 + TLS
- 開發環境：允許 HTTP (`localhost:3000`)，CI/CD 測試環境同

### 6.2 HTTP 安全 Headers

由 Express Middleware (`helmet`) 統一注入：

- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `X-XSS-Protection: 1; mode=block`
- `Strict-Transport-Security: max-age=31536000` (啟用 HTTPS 時)

### 6.3 速率限制 (Rate Limiting)

| 端點 | 限制 | 說明 |
| :--- | :--- | :--- |
| `POST /auth/login` | 5 次/分鐘/IP | 防暴力破解 |
| `POST /transactions/voice` | 30 次/分鐘/User | 語音辨識成本控制 |
| `POST /sync/push` | 10 次/分鐘/User | 批量同步頻率控制 |
| `GET /sync/pull` | 30 次/分鐘/User | 增量拉取頻率控制 |
| 其他所有端點 | 200 次/分鐘/User | 一般使用保護 |

超出限制回應 `429 Too Many Requests`，並在 Header 中回傳 `Retry-After` 秒數。

### 6.4 輸入驗證安全

- 所有輸入經 `express-validator` 或 Zod 驗證
- SQL Injection 防護：Prisma ORM 參數化查詢
- XSS 防護：輸出 JSON 不含 HTML，前端負責 escape
- 檔案上傳限制：照片最大 5MB, 僅允許 image/jpeg, image/png；音檔最大 10MB, 僅允許 audio/wav, audio/m4a

---

## 7. API 端點詳述 (API Endpoint Definitions)

### 7.1 資源：認證 (Auth)

#### `POST /api/v1/auth/login`

- **描述：** 使用者登入，回傳 JWT Token
- **授權：** Public (不需 Token)
- **請求體：** `LoginRequest`

```json
{
  "email": "owner@coffeeshop.local",
  "password": "your_password"
}
```

- **成功回應 (200 OK)：** `LoginResponse`

```json
{
  "data": {
    "token": "eyJhbG...",
    "user": {
      "id": "usr_01HXYZ",
      "name": "店主",
      "email": "owner@coffeeshop.local",
      "role": "owner",
      "created_at": "2026-01-15T08:00:00Z"
    }
  }
}
```

- **錯誤回應：**
  - `401` — `authentication_failed`: 帳號或密碼錯誤
  - `400` — `parameter_missing`: 缺少 email 或 password
  - `429` — `rate_limit_exceeded`: 登入嘗試過頻

---

### 7.2 資源：使用者 (Users)

#### `POST /api/v1/users`

- **描述：** 新增員工帳號
- **授權：** `owner`
- **對應 BDD：** US-401
- **請求體：** `UserCreateRequest`

```json
{
  "name": "王小明",
  "email": "xiaoming@example.com",
  "password": "Temp1234!",
  "role": "employee"
}
```

- **成功回應 (201 Created)：** `User`
- **錯誤回應：**
  - `409` — `email_already_exists`
  - `422` — `account_limit_exceeded` (已達 10 個帳號上限)
  - `400` — `parameter_missing` / `parameter_invalid`
  - `403` — `permission_denied` (非 owner)

#### `GET /api/v1/users`

- **描述：** 列出所有團隊成員
- **授權：** `owner`
- **成功回應 (200 OK)：** `User[]`

#### `PATCH /api/v1/users/{user_id}`

- **描述：** 修改員工資料或角色 (例如 employee → manager)
- **授權：** `owner`
- **對應 BDD：** US-402
- **請求體 (部分更新)：**

```json
{
  "role": "manager"
}
```

- **成功回應 (200 OK)：** 更新後的 `User`
- **錯誤回應：**
  - `404` — `resource_not_found`
  - `403` — `permission_denied`

#### `DELETE /api/v1/users/{user_id}`

- **描述：** 刪除員工帳號 (Soft Delete)
- **授權：** `owner`
- **成功回應：** `204 No Content`
- **業務規則：** 店主帳號不可刪除自己

#### `GET /api/v1/users/me`

- **描述：** 取得當前登入使用者資訊
- **授權：** 所有已認證使用者
- **成功回應 (200 OK)：** `User`

---

### 7.3 資源：交易記錄 (Transactions)

#### `POST /api/v1/transactions`

- **描述：** 新增一筆交易記錄 (手動輸入 / 快速按鈕)
- **授權：** `employee`, `manager`, `owner`
- **對應 BDD：** US-101, US-102, US-103
- **請求體：** `TransactionCreateRequest`

```json
{
  "type": "income",
  "category": "營收",
  "item_name": "美式咖啡",
  "amount": 360,
  "quantity": 3,
  "note": "",
  "quick_button_id": "qb_01HABC"
}
```

**備註：**
- `quick_button_id` 為選填，提供時從快速按鈕帶入預設值
- `type` 為 `income` 或 `expense`
- `category` 可由前端傳入 (手動選擇) 或為空 (交由 AI 分類端點)
- 系統自動記錄 `user_id` (記錄者) 與 `created_at`

- **成功回應 (201 Created)：** `Transaction`
- **錯誤回應：**
  - `400` — `parameter_missing` (缺少 amount / type)
  - `422` — `quantity_invalid` (quantity ≤ 0)

#### `GET /api/v1/transactions`

- **描述：** 查詢交易列表 (支援篩選、排序、分頁)
- **授權：** `employee`, `manager`, `owner`
- **對應 BDD：** US-403 (篩選記錄者)
- **查詢參數：**

| 參數 | 類型 | 說明 |
| :--- | :--- | :--- |
| `type` | `income` / `expense` | 篩選交易類型 |
| `category` | string | 篩選分類 |
| `user_id` | string | 篩選記錄者 (對應 US-403) |
| `start_date` | `YYYY-MM-DD` | 起始日期 |
| `end_date` | `YYYY-MM-DD` | 結束日期 |
| `sort_by` | string | 排序欄位，預設 `-created_at` |
| `page` | integer | 頁碼，預設 `1` |
| `page_size` | integer | 每頁筆數，預設 `20` |

- **可排序欄位：** `created_at`, `amount`, `item_name`
- **成功回應 (200 OK)：** `Transaction[]` + `pagination`

#### `GET /api/v1/transactions/{transaction_id}`

- **描述：** 取得單筆交易詳情 (含附件照片 URL)
- **授權：** `employee`, `manager`, `owner`
- **成功回應 (200 OK)：** `Transaction` (含 `photo` 物件)

#### `PATCH /api/v1/transactions/{transaction_id}`

- **描述：** 修改交易記錄 (例如修正分類、金額)
- **授權：** 記錄者本人 或 `manager` / `owner`
- **請求體 (部分更新)：**

```json
{
  "category": "設備維修",
  "amount": 500,
  "note": "修正分類"
}
```

- **成功回應 (200 OK)：** 更新後的 `Transaction`

#### `DELETE /api/v1/transactions/{transaction_id}`

- **描述：** 刪除交易記錄
- **授權：** `manager`, `owner`
- **成功回應：** `204 No Content`

---

### 7.4 資源：語音輸入 (Voice)

#### `POST /api/v1/transactions/voice`

- **描述：** 上傳音檔 → STT 辨識 → AI 分類 → 回傳建議結果 (不直接儲存)
- **授權：** `employee`, `manager`, `owner`
- **對應 BDD：** US-101
- **Content-Type：** `multipart/form-data`
- **請求欄位：**

| 欄位 | 類型 | 必填 | 說明 |
| :--- | :--- | :--- | :--- |
| `audio` | file | 是 | 音檔 (.wav / .m4a)，最大 10MB |

- **成功回應 (200 OK)：** `VoiceParsedResult`

```json
{
  "data": {
    "voice_text": "買咖啡豆 500 元",
    "parsed": {
      "type": "expense",
      "category": "進貨成本",
      "item_name": "咖啡豆",
      "amount": 500,
      "confidence": 0.92
    }
  }
}
```

- **錯誤回應：**
  - `422` — `voice_recognition_failed`: 無法辨識語音
  - `503` — `network_required`: 無網路連線

**使用流程：** 前端呼叫此端點取得建議 → 顯示確認畫面 → 使用者確認後呼叫 `POST /api/v1/transactions` 正式儲存。

---

#### `POST /api/v1/transactions/classify`

- **描述：** 將文字描述傳送至 AI 分類 → 回傳分類建議
- **授權：** `employee`, `manager`, `owner`
- **對應 BDD：** US-103
- **請求體：**

```json
{
  "description": "繳電費 800 元"
}
```

- **成功回應 (200 OK)：**

```json
{
  "data": {
    "category": "固定支出",
    "item_name": "電費",
    "amount": 800,
    "source": "gemini",
    "confidence": 0.95
  }
}
```

- **Fallback 行為：** 若 Gemini API 不可用，自動使用本地關鍵字分類，`source` 欄位回傳 `"local_keyword"`
- **錯誤回應：**
  - `422` — `classification_failed`: AI 與本地規則皆無法分類

---

### 7.5 資源：快速按鈕 (Quick Buttons)

#### `GET /api/v1/quick-buttons`

- **描述：** 列出所有快速按鈕 (依 `display_order` 升序)
- **授權：** `employee`, `manager`, `owner`
- **成功回應 (200 OK)：** `QuickButton[]`

#### `POST /api/v1/quick-buttons`

- **描述：** 新增快速按鈕
- **授權：** `owner`
- **對應 BDD：** US-105
- **Content-Type：** `multipart/form-data` (含選填圖片)
- **請求欄位：**

| 欄位 | 類型 | 必填 | 說明 |
| :--- | :--- | :--- | :--- |
| `item_name` | string | 是 | 品項名稱 |
| `default_price` | integer | 是 | 預設價格 (元) |
| `icon` | file | 否 | 品項圖片 (max 2MB, jpeg/png) |

- **成功回應 (201 Created)：** `QuickButton`
- **錯誤回應：**
  - `400` — `parameter_missing`: 品項名稱為必填

#### `PATCH /api/v1/quick-buttons/{button_id}`

- **描述：** 修改快速按鈕 (名稱、價格、圖片)
- **授權：** `owner`
- **成功回應 (200 OK)：** 更新後的 `QuickButton`

#### `DELETE /api/v1/quick-buttons/{button_id}`

- **描述：** 刪除快速按鈕
- **授權：** `owner`
- **成功回應：** `204 No Content`

#### `PATCH /api/v1/quick-buttons/reorder`

- **描述：** 批量更新按鈕顯示順序
- **授權：** `owner`
- **請求體：**

```json
{
  "order": [
    { "id": "qb_03", "display_order": 1 },
    { "id": "qb_01", "display_order": 2 },
    { "id": "qb_02", "display_order": 3 }
  ]
}
```

- **成功回應 (200 OK)：** 更新後的 `QuickButton[]`

---

### 7.6 資源：儀表板 (Dashboard)

#### `GET /api/v1/dashboard/today`

- **描述：** 今日營收概覽 (營收/杯數/淨利/週同比)
- **授權：** `owner`, `manager`
- **對應 BDD：** US-201
- **成功回應 (200 OK)：**

```json
{
  "data": {
    "date": "2026-02-11",
    "day_of_week": "Wednesday",
    "total_income": 2400,
    "total_expense": 500,
    "net_profit": 1900,
    "total_cups": 18,
    "items_breakdown": [
      { "item_name": "美式咖啡", "quantity": 10, "revenue": 1200 },
      { "item_name": "拿鐵咖啡", "quantity": 8, "revenue": 1200 }
    ],
    "wow_comparison": {
      "last_week_date": "2026-02-04",
      "last_week_income": 2000,
      "difference": 400,
      "percentage_change": 20.0,
      "direction": "up"
    }
  }
}
```

**備註：** 若無上週同天資料，`wow_comparison` 為 `null`。

#### `GET /api/v1/dashboard/trend`

- **描述：** 營收趨勢折線圖資料
- **授權：** `owner`, `manager`
- **對應 BDD：** US-202
- **查詢參數：**

| 參數 | 類型 | 預設 | 說明 |
| :--- | :--- | :--- | :--- |
| `days` | `7` / `30` | `7` | 趨勢期間 |

- **成功回應 (200 OK)：**

```json
{
  "data": {
    "period_days": 7,
    "trend": [
      { "date": "2026-02-05", "income": 1800, "expense": 600, "net_profit": 1200 },
      { "date": "2026-02-06", "income": 2200, "expense": 400, "net_profit": 1800 },
      ...
    ]
  }
}
```

#### `GET /api/v1/dashboard/ranking/today`

- **描述：** 今日品項銷售排行
- **授權：** `owner`, `manager`
- **對應 BDD：** US-203
- **查詢參數：**

| 參數 | 類型 | 預設 | 說明 |
| :--- | :--- | :--- | :--- |
| `limit` | integer | `3` | Top N 排行 (設為 `0` 查看全部) |

- **成功回應 (200 OK)：**

```json
{
  "data": {
    "date": "2026-02-11",
    "ranking": [
      { "rank": 1, "item_name": "美式咖啡", "quantity": 15, "revenue": 1800, "percentage": 39.5 },
      { "rank": 2, "item_name": "拿鐵咖啡", "quantity": 12, "revenue": 1800, "percentage": 31.6 },
      { "rank": 3, "item_name": "卡布奇諾", "quantity": 8, "revenue": 1120, "percentage": 17.5 }
    ]
  }
}
```

---

### 7.7 資源：月報表 (Reports)

#### `GET /api/v1/reports/monthly`

- **描述：** 列出所有可用的月報表
- **授權：** `owner`, `manager`
- **成功回應 (200 OK)：**

```json
{
  "data": [
    { "period": "2026-02", "generated_at": "2026-03-01T08:00:00Z" },
    { "period": "2026-01", "generated_at": "2026-02-01T08:00:00Z" }
  ]
}
```

#### `GET /api/v1/reports/monthly/{period}`

- **描述：** 取得指定月份的完整報表 (例如 `2026-02`)
- **授權：** `owner`, `manager`
- **對應 BDD：** US-301, US-302, US-303, US-304
- **成功回應 (200 OK)：**

```json
{
  "data": {
    "period": "2026-02",
    "generated_at": "2026-03-01T08:00:00Z",
    "summary": {
      "total_income": 85000,
      "total_expense": 41000,
      "net_profit": 44000,
      "prev_month_net_profit": 40000,
      "mom_change_percent": 10.0,
      "mom_direction": "up"
    },
    "cost_breakdown": [
      {
        "category": "進貨成本",
        "amount": 20000,
        "percentage": 48.8,
        "prev_month_amount": 18000,
        "change_percent": 11.1
      },
      {
        "category": "固定支出",
        "amount": 18000,
        "percentage": 43.9,
        "prev_month_amount": 18000,
        "change_percent": 0.0
      },
      {
        "category": "設備維修",
        "amount": 3000,
        "percentage": 7.3,
        "prev_month_amount": 4000,
        "change_percent": -25.0
      }
    ],
    "top_items": [
      { "rank": 1, "item_name": "美式咖啡", "quantity": 320, "revenue": 38400, "percentage": 45.2 },
      { "rank": 2, "item_name": "拿鐵咖啡", "quantity": 250, "revenue": 37500, "percentage": 44.1 }
    ],
    "ranking_mode": "quantity"
  }
}
```

- **錯誤回應：**
  - `422` — `report_not_ready`: 該月報表尚未產生
  - `404` — `resource_not_found`: 系統啟用前的月份

---

### 7.8 資源：系統健康 (Health)

#### `GET /api/v1/health`

- **描述：** 系統健康檢查
- **授權：** Public (不需 Token)
- **成功回應 (200 OK)：**

```json
{
  "status": "ok",
  "version": "1.0.0",
  "database": "connected",
  "uptime_seconds": 86400,
  "disk_usage_percent": 45.2
}
```

### 7.9 資源：離線同步 (Sync)

> 對齊 PRD NFR：「可離線記帳，網路恢復後自動同步」

#### `POST /api/v1/sync/push`

- **描述：** 批量上傳離線期間建立/修改/刪除的記錄
- **授權：** 所有已認證使用者
- **請求體：** `SyncPushRequest`

```json
{
  "changes": [
    {
      "client_id": "550e8400-e29b-41d4-a716-446655440000",
      "resource": "transaction",
      "action": "create",
      "data": {
        "type": "income",
        "category": "營收",
        "item_name": "美式咖啡",
        "amount": 120,
        "quantity": 1
      },
      "client_timestamp": "2026-02-11T08:30:00Z"
    },
    {
      "client_id": "550e8400-e29b-41d4-a716-446655440001",
      "resource": "transaction",
      "action": "update",
      "resource_id": "tx_01HABC",
      "data": {
        "category": "設備維修",
        "note": "修正分類"
      },
      "client_timestamp": "2026-02-11T08:31:00Z"
    }
  ]
}
```

- **成功回應 (200 OK)：** `SyncPushResponse`

```json
{
  "data": {
    "results": [
      {
        "client_id": "550e8400-e29b-41d4-a716-446655440000",
        "status": "accepted",
        "server_id": "tx_01HXYZ",
        "server_timestamp": "2026-02-11T08:30:01Z"
      },
      {
        "client_id": "550e8400-e29b-41d4-a716-446655440001",
        "status": "conflict",
        "server_id": "tx_01HABC",
        "server_timestamp": "2026-02-11T08:25:00Z",
        "message": "此記錄已被其他使用者修改，伺服器版本較新"
      }
    ],
    "accepted_count": 1,
    "conflict_count": 1,
    "error_count": 0
  }
}
```

- **錯誤回應：**
  - `400` — `parameter_invalid`: changes 陣列為空或超過上限 (最大 100 筆/次)
  - `422` — `sync_conflict`: 衝突明細在回應 `results` 中逐筆列出

---

#### `GET /api/v1/sync/pull`

- **描述：** 拉取自上次同步以來伺服器端所有變更 (含其他裝置/使用者的操作)
- **授權：** 所有已認證使用者
- **查詢參數：**

| 參數 | 類型 | 必填 | 說明 |
| :--- | :--- | :--- | :--- |
| `since` | ISO 8601 | 是 | 上次同步時間戳 (首次同步傳 `1970-01-01T00:00:00Z`) |
| `resource` | string | 否 | 限定資源類型：`transaction`, `quick_button` (不傳則全部) |
| `limit` | integer | 否 | 每次拉取上限，預設 `200`，最大 `500` |

- **成功回應 (200 OK)：** `SyncPullResponse`

```json
{
  "data": {
    "changes": [
      {
        "resource": "transaction",
        "action": "create",
        "server_id": "tx_01HDEF",
        "data": {
          "id": "tx_01HDEF",
          "type": "expense",
          "category": "進貨成本",
          "item_name": "牛奶",
          "amount": 300,
          "quantity": 2,
          "user_id": "usr_02ABC",
          "user_name": "王小明",
          "created_at": "2026-02-11T09:00:00Z",
          "updated_at": "2026-02-11T09:00:00Z"
        },
        "server_timestamp": "2026-02-11T09:00:00Z"
      }
    ],
    "sync_token": "2026-02-11T09:00:00Z",
    "has_more": false
  }
}
```

**使用流程：**
1. 前端儲存 `sync_token`，下次請求帶入 `since` 參數
2. 若 `has_more` 為 `true`，以回傳的 `sync_token` 繼續拉取，直到 `has_more` 為 `false`
3. 前端將拉取到的變更合併至本地資料庫

---

#### `GET /api/v1/sync/status`

- **描述：** 查詢當前裝置的同步狀態
- **授權：** 所有已認證使用者
- **成功回應 (200 OK)：**

```json
{
  "data": {
    "last_sync_at": "2026-02-11T08:30:01Z",
    "server_now": "2026-02-11T09:15:00Z",
    "pending_changes_count": 3
  }
}
```

---

## 8. 資料模型/Schema 定義 (Data Models / Schema Definitions)

### 8.1 `User`

```json
{
  "id": "string (usr_...)",
  "name": "string",
  "email": "string (email format)",
  "role": "owner | manager | employee",
  "created_at": "string (ISO 8601)",
  "updated_at": "string (ISO 8601)"
}
```

### 8.2 `LoginRequest`

```json
{
  "email": "string (required)",
  "password": "string (required)"
}
```

### 8.3 `UserCreateRequest`

```json
{
  "name": "string (required, max 50)",
  "email": "string (required, email format)",
  "password": "string (required, min 8 chars)",
  "role": "employee | manager (required)"
}
```

### 8.4 `Transaction`

```json
{
  "id": "string (tx_...)",
  "user_id": "string (記錄者 ID)",
  "user_name": "string (記錄者姓名)",
  "type": "income | expense",
  "category": "string (營收 | 進貨成本 | 固定支出 | 設備維修 | 未分類)",
  "item_name": "string",
  "amount": "integer (元)",
  "quantity": "integer (預設 1)",
  "note": "string | null",
  "voice_text": "string | null (語音辨識原文)",
  "is_ai_classified": "boolean",
  "photo": {
    "id": "string (photo_...)",
    "url": "string (本地檔案 URL)",
    "mime_type": "string"
  },
  "created_at": "string (ISO 8601)",
  "updated_at": "string (ISO 8601)"
}
```

### 8.5 `TransactionCreateRequest`

```json
{
  "type": "income | expense (required)",
  "category": "string (選填, 未提供時為 '未分類')",
  "item_name": "string (required)",
  "amount": "integer (required, > 0)",
  "quantity": "integer (選填, 預設 1, > 0)",
  "note": "string (選填)",
  "voice_text": "string (選填, 語音原文)",
  "is_ai_classified": "boolean (選填, 預設 false)",
  "quick_button_id": "string (選填)"
}
```

### 8.6 `QuickButton`

```json
{
  "id": "string (qb_...)",
  "item_name": "string",
  "default_price": "integer (元)",
  "icon_url": "string | null",
  "display_order": "integer",
  "created_at": "string (ISO 8601)",
  "updated_at": "string (ISO 8601)"
}
```

### 8.7 `VoiceParsedResult`

```json
{
  "voice_text": "string (STT 辨識文字)",
  "parsed": {
    "type": "income | expense",
    "category": "string",
    "item_name": "string",
    "amount": "integer",
    "confidence": "number (0-1)"
  }
}
```

### 8.8 `MonthlyReport`

```json
{
  "period": "string (YYYY-MM)",
  "generated_at": "string (ISO 8601)",
  "summary": { "total_income", "total_expense", "net_profit", "prev_month_net_profit", "mom_change_percent", "mom_direction" },
  "cost_breakdown": [{ "category", "amount", "percentage", "prev_month_amount", "change_percent" }],
  "top_items": [{ "rank", "item_name", "quantity", "revenue", "percentage" }],
  "ranking_mode": "quantity | revenue"
}
```

### 8.9 `SyncChangeItem` (Push 單筆)

```json
{
  "client_id": "string (UUID v4, required — 前端產生的唯一識別)",
  "resource": "transaction | quick_button (required)",
  "action": "create | update | delete (required)",
  "resource_id": "string (update/delete 時必填，對應伺服器端 ID)",
  "data": "object (create/update 時必填，欄位同對應的 CreateRequest)",
  "client_timestamp": "string (ISO 8601, required — 前端操作時間)"
}
```

### 8.10 `SyncPushRequest`

```json
{
  "changes": "SyncChangeItem[] (required, min 1, max 100)"
}
```

### 8.11 `SyncPushResponse`

```json
{
  "results": [
    {
      "client_id": "string (UUID v4)",
      "status": "accepted | conflict | error",
      "server_id": "string (伺服器端 ID)",
      "server_timestamp": "string (ISO 8601)",
      "message": "string | null (conflict/error 時的說明)"
    }
  ],
  "accepted_count": "integer",
  "conflict_count": "integer",
  "error_count": "integer"
}
```

### 8.12 `SyncPullResponse`

```json
{
  "changes": [
    {
      "resource": "transaction | quick_button",
      "action": "create | update | delete",
      "server_id": "string",
      "data": "object (對應資源的完整 Schema；delete 時為 null)",
      "server_timestamp": "string (ISO 8601)"
    }
  ],
  "sync_token": "string (ISO 8601 — 下次請求帶入 since)",
  "has_more": "boolean"
}
```

---

## 9. API 生命週期與版本控制 (API Lifecycle and Versioning)

### 9.1 API 生命週期階段

| 階段 | 對應時程 | 說明 |
| :--- | :--- | :--- |
| **Development** | 2026-03-05 ~ 2026-03-20 | 內部開發，API 極不穩定 |
| **Alpha** | 2026-03-21 ~ 2026-04-01 | 整合測試，前後端聯調 |
| **Beta** | 2026-04-02 ~ 2026-04-14 | 店主驗收測試 |
| **GA (v1)** | 2026-04-15 | MVP 正式上線 |
| **GA (v1 完整版)** | 2026-06-30 | 完整版功能全部上線 |

### 9.2 版本控制策略

- **策略：** URL 路徑版本控制 (`/api/v1/...`)
- **向後兼容變更 (不改版)：** 新增端點、新增回應欄位、新增選填參數
- **破壞性變更 (升版至 v2)：** 刪除/重命名欄位、修改型別、新增必填參數

### 9.3 棄用策略

由於本系統為單店專屬部署 (前後端同步更新)，棄用策略簡化為：

1. 在 Release Note 中標註破壞性變更
2. 前後端同步部署新版本
3. 無需長期維護多版本 API

---

## 10. 附錄 (Appendix)

### 10.1 端點總覽速查表

| Method | Endpoint | 說明 | 授權 |
| :--- | :--- | :--- | :--- |
| POST | `/auth/login` | 登入 | Public |
| POST | `/users` | 新增員工 | Owner |
| GET | `/users` | 列出成員 | Owner |
| PATCH | `/users/:id` | 修改員工 | Owner |
| DELETE | `/users/:id` | 刪除員工 | Owner |
| GET | `/users/me` | 當前使用者 | Auth |
| POST | `/transactions` | 新增交易 | Employee+ |
| GET | `/transactions` | 查詢交易 | Employee+ |
| GET | `/transactions/:id` | 交易詳情 | Employee+ |
| PATCH | `/transactions/:id` | 修改交易 | 記錄者/Manager+ |
| DELETE | `/transactions/:id` | 刪除交易 | Manager+ |
| POST | `/transactions/voice` | 語音記帳 | Employee+ |
| POST | `/transactions/classify` | AI 分類 | Employee+ |
| GET | `/quick-buttons` | 列出按鈕 | Employee+ |
| POST | `/quick-buttons` | 新增按鈕 | Owner |
| PATCH | `/quick-buttons/:id` | 修改按鈕 | Owner |
| DELETE | `/quick-buttons/:id` | 刪除按鈕 | Owner |
| PATCH | `/quick-buttons/reorder` | 排序按鈕 | Owner |
| GET | `/dashboard/today` | 今日概覽 | Owner/Manager |
| GET | `/dashboard/trend` | 營收趨勢 | Owner/Manager |
| GET | `/dashboard/ranking/today` | 今日排行 | Owner/Manager |
| GET | `/reports/monthly` | 月報表列表 | Owner/Manager |
| GET | `/reports/monthly/:period` | 月報表詳情 | Owner/Manager |
| GET | `/health` | 健康檢查 | Public |
| POST | `/sync/push` | 離線資料上傳 | Auth |
| GET | `/sync/pull` | 增量拉取變更 | Auth |
| GET | `/sync/status` | 同步狀態查詢 | Auth |

### 10.2 照片附件上傳範例

```bash
curl --request POST \
  --url https://192.168.1.100/api/v1/transactions \
  --header 'Authorization: Bearer eyJhbG...' \
  --form 'type=expense' \
  --form 'category=進貨成本' \
  --form 'item_name=咖啡豆' \
  --form 'amount=500' \
  --form 'photo=@/path/to/receipt.jpg'
```

---

**文件審核記錄 (Review History):**

| 日期 | 審核人 | 版本 | 變更摘要 |
| :--- | :--- | :--- | :--- |
| 2026-02-11 | [Backend Engineer] | v1.0.0 | 初稿完成，涵蓋 24 個端點、8 個 Schema |
| 2026-02-11 | [Backend Engineer] | v1.1.0 | 對齊 PRD：(1) HTTPS + TLS 1.3 強制啟用 (2) 新增離線同步 API (3 端點 + 4 Schema) |
