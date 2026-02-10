# 模組規格與測試案例 (Module Specification & Test Cases) - CoffeeBook AI

---

**文件版本 (Document Version):** `v1.0`
**最後更新 (Last Updated):** `2026-02-11`
**主要作者 (Lead Author):** `[Backend Engineer]`
**審核者 (Reviewers):** `[Lead Engineer, QA Lead]`
**狀態 (Status):** `待開發 (To Do)`
**對應架構文檔:** `05_architecture_design.md`
**對應 BDD 文件:** `03_bdd_scenarios.md`

**相關文件：**
- 系統架構與模組職責 → [架構設計文件 (05_architecture_design.md)](05_architecture_design.md)
- BDD 行為場景 (本文件的上層情境來源) → [BDD 情境文件 (03_bdd_scenarios.md)](03_bdd_scenarios.md)
- API 端點定義 (本文件的模組對應的 HTTP 介面) → [API 設計規範 (06_api_design_specification.md)](06_api_design_specification.md)
- 使用者故事 → [PRD (02_prd.md)](02_prd.md)

---

## 目錄 (Table of Contents)

- [模組: AuthService](#模組-authservice)
  - [規格 1: login](#規格-1-loginemail-string-password-string-promiseloginresult)
  - [規格 2: createUser](#規格-2-createuserdata-usercreateinput-promiseuser)
- [模組: TransactionService](#模組-transactionservice)
  - [規格 3: createTransaction](#規格-3-createtransactiondata-txcreateinput-userid-string-promisetransaction)
  - [規格 4: createFromQuickButton](#規格-4-createfromquickbuttonbuttonid-string-overrides-partial-userid-string-promisetransaction)
- [模組: AIClassifierAdapter](#模組-aiclassifieradapter)
  - [規格 5: classifyExpense](#規格-5-classifyexpensedescription-string-promiseclassificationresult)
- [模組: VoiceProcessor](#模組-voiceprocessor)
  - [規格 6: processVoiceInput](#規格-6-processvoiceinputaudiobuffer-buffer-promisevoiceparsedresult)
- [模組: QuickButtonService](#模組-quickbuttonservice)
  - [規格 7: reorderButtons](#規格-7-reorderbuttonsorder-reorderinput-promisevoid)
- [模組: ReportGenerator](#模組-reportgenerator)
  - [規格 8: generateMonthlyReport](#規格-8-generatemonthlyreportperiod-string-promisemonthlyreport)
  - [規格 9: getDailyOverview](#規格-9-getdailyoverviewdate-date-promisedailyoverview)

---

**目的**: 本文件將 CoffeeBook AI 的 BDD 情境分解到具體模組層級，定義每個核心函式的契約式設計 (Design by Contract, DbC)，並以 Arrange-Act-Assert 格式列出測試案例，直接指導 TDD 實踐。

---

## 模組: AuthService

**對應架構文件:** `05_architecture_design.md#auth-module`
**對應 BDD Feature:** `epic_user_management.feature` (US-401, US-402)

---

### 規格 1: `login(email: string, password: string): Promise<LoginResult>`

**描述:** 驗證使用者帳號密碼，成功則簽發 JWT Token。

**契約式設計 (DbC):**

- **前置條件 (Preconditions):**
  1. `email` 不可為空，且必須為有效的 Email 格式
  2. `password` 不可為空
- **後置條件 (Postconditions):**
  1. 回傳的 `token` 為有效的 JWT，包含 `user_id`, `role`, `exp` (24h)
  2. 回傳的 `user` 物件不包含 `password_hash`
- **不變性 (Invariants):**
  1. 密碼永遠不以明文形式出現在日誌或回應中
  2. 無論帳號是否存在，失敗回應時間應一致 (防止 Timing Attack)

---

### 測試情境與案例 (Test Scenarios & Cases)

#### 情境 1: 正常路徑 (Happy Path)

- **測試案例 ID:** `TC-Auth-001`
- **描述:** 使用正確帳號密碼成功登入
- **測試步驟:**
  1. **Arrange:** 建立使用者 `{ email: "owner@test.com", password: "Pass1234!", role: "owner" }`，密碼以 bcrypt 雜湊儲存
  2. **Act:** 呼叫 `authService.login("owner@test.com", "Pass1234!")`
  3. **Assert:**
     - 回傳 `token` 為非空字串
     - JWT 解碼後 `payload.user_id` 與使用者 ID 一致
     - JWT 解碼後 `payload.role` 為 `"owner"`
     - 回傳 `user.email` 為 `"owner@test.com"`
     - 回傳 `user` 不含 `password_hash` 欄位

#### 情境 2: 無效密碼 (Sad Path)

- **測試案例 ID:** `TC-Auth-002`
- **描述:** 使用錯誤密碼登入失敗
- **測試步驟:**
  1. **Arrange:** 同 TC-Auth-001
  2. **Act:** 呼叫 `authService.login("owner@test.com", "WrongPass!")`
  3. **Assert:**
     - 拋出 `AuthenticationError`，code 為 `"authentication_failed"`
     - 錯誤訊息為 `"Invalid email or password"`

#### 情境 3: 帳號不存在 (Sad Path)

- **測試案例 ID:** `TC-Auth-003`
- **描述:** 使用不存在的 Email 登入
- **測試步驟:**
  1. **Arrange:** 資料庫中無此 Email
  2. **Act:** 呼叫 `authService.login("nobody@test.com", "Pass1234!")`
  3. **Assert:**
     - 拋出 `AuthenticationError`，code 為 `"authentication_failed"`
     - 錯誤訊息與 TC-Auth-002 **相同** (不洩漏帳號是否存在)

#### 情境 4: 無效輸入 (Edge Case)

- **測試案例 ID:** `TC-Auth-004`
- **描述:** Email 或密碼為空
- **測試步驟:**
  1. **Arrange:** 無
  2. **Act:** 呼叫 `authService.login("", "Pass1234!")`
  3. **Assert:** 拋出 `ValidationError`，code 為 `"parameter_missing"`，param 為 `"email"`

---

### 規格 2: `createUser(data: UserCreateInput): Promise<User>`

**描述:** 店主新增一個員工帳號。

**契約式設計 (DbC):**

- **前置條件 (Preconditions):**
  1. `data.name` 不可為空，最大 50 字元
  2. `data.email` 不可為空，且必須為有效 Email 格式
  3. `data.password` 不可為空，最小 8 字元
  4. `data.role` 必須為 `"employee"` 或 `"manager"`
  5. 目前系統帳號總數 < 10
  6. `data.email` 在系統中不存在
- **後置條件 (Postconditions):**
  1. 資料庫新增一筆 `users` 記錄
  2. `password` 以 bcrypt 雜湊儲存，salt rounds ≥ 10
  3. 回傳的 `User` 物件不包含 `password_hash`
- **不變性 (Invariants):**
  1. 系統帳號總數永遠 ≤ 10

---

### 測試情境與案例 (Test Scenarios & Cases)

#### 情境 1: 正常路徑 (Happy Path)

- **測試案例 ID:** `TC-User-001`
- **描述:** 成功新增一名員工
- **測試步驟:**
  1. **Arrange:** 系統中有 1 個 owner 帳號
  2. **Act:** 呼叫 `authService.createUser({ name: "王小明", email: "xm@test.com", password: "Temp1234!", role: "employee" })`
  3. **Assert:**
     - 回傳 `user.name` 為 `"王小明"`
     - 回傳 `user.role` 為 `"employee"`
     - 資料庫 `users` 表記錄數為 2
     - 資料庫中 `password_hash` 不等於 `"Temp1234!"` (已雜湊)

#### 情境 2: Email 重複 (Sad Path)

- **測試案例 ID:** `TC-User-002`
- **描述:** 使用已存在的 Email 新增員工
- **測試步驟:**
  1. **Arrange:** 系統中已存在 `email: "xm@test.com"` 的帳號
  2. **Act:** 呼叫 `authService.createUser({ name: "另一人", email: "xm@test.com", password: "Temp1234!", role: "employee" })`
  3. **Assert:** 拋出 `ConflictError`，code 為 `"email_already_exists"`

#### 情境 3: 帳號上限 (Business Rule)

- **測試案例 ID:** `TC-User-003`
- **描述:** 帳號數量達上限 (10) 時新增帳號
- **測試步驟:**
  1. **Arrange:** 系統中已有 10 個帳號 (1 owner + 9 employees)
  2. **Act:** 呼叫 `authService.createUser({ name: "第11人", email: "new@test.com", password: "Temp1234!", role: "employee" })`
  3. **Assert:** 拋出 `BusinessRuleError`，code 為 `"account_limit_exceeded"`

#### 情境 4: 無效輸入 (Edge Case)

- **測試案例 ID:** `TC-User-004`
- **描述:** 密碼不足 8 字元
- **測試步驟:**
  1. **Arrange:** 無
  2. **Act:** 呼叫 `authService.createUser({ name: "王小明", email: "xm@test.com", password: "123", role: "employee" })`
  3. **Assert:** 拋出 `ValidationError`，code 為 `"parameter_invalid"`，param 為 `"password"`

---

## 模組: TransactionService

**對應架構文件:** `05_architecture_design.md#transaction-module`
**對應 BDD Feature:** `epic_smart_input.feature` (US-101, US-102, US-103)

---

### 規格 3: `createTransaction(data: TxCreateInput, userId: string): Promise<Transaction>`

**描述:** 新增一筆交易記錄，自動標記記錄者。

**契約式設計 (DbC):**

- **前置條件 (Preconditions):**
  1. `data.type` 必須為 `"income"` 或 `"expense"`
  2. `data.amount` 必須為正整數 (> 0)
  3. `data.quantity` 若提供，必須為正整數 (> 0)
  4. `data.item_name` 不可為空
  5. `userId` 必須對應一個存在的使用者
- **後置條件 (Postconditions):**
  1. 資料庫新增一筆 `transactions` 記錄
  2. `transaction.user_id` 等於傳入的 `userId`
  3. `transaction.created_at` 為當下時間
  4. 若 `data.quantity` 未提供，預設為 1
  5. 若 `data.category` 未提供，預設為 `"未分類"`
- **不變性 (Invariants):**
  1. `amount` 永遠 > 0
  2. `quantity` 永遠 ≥ 1

---

### 測試情境與案例 (Test Scenarios & Cases)

#### 情境 1: 正常路徑 — 手動記帳 (Happy Path)

- **測試案例 ID:** `TC-Tx-001`
- **描述:** 手動輸入一筆支出
- **測試步驟:**
  1. **Arrange:** 使用者 `usr_01` 存在
  2. **Act:** 呼叫 `transactionService.createTransaction({ type: "expense", category: "進貨成本", item_name: "咖啡豆", amount: 500, note: "咖啡豆" }, "usr_01")`
  3. **Assert:**
     - 回傳 `transaction.type` 為 `"expense"`
     - 回傳 `transaction.amount` 為 `500`
     - 回傳 `transaction.user_id` 為 `"usr_01"`
     - 回傳 `transaction.quantity` 為 `1` (預設)

#### 情境 2: 正常路徑 — 含數量 (Happy Path)

- **測試案例 ID:** `TC-Tx-002`
- **描述:** 記錄 3 杯美式咖啡銷售
- **測試步驟:**
  1. **Arrange:** 使用者存在
  2. **Act:** 呼叫 `createTransaction({ type: "income", category: "營收", item_name: "美式咖啡", amount: 360, quantity: 3 }, "usr_01")`
  3. **Assert:**
     - `transaction.quantity` 為 `3`
     - `transaction.amount` 為 `360`

#### 情境 3: 無效輸入 — 金額為 0 (Invalid Input)

- **測試案例 ID:** `TC-Tx-003`
- **描述:** 金額為 0 時應拒絕
- **測試步驟:**
  1. **Arrange:** 無
  2. **Act:** 呼叫 `createTransaction({ type: "expense", item_name: "test", amount: 0 }, "usr_01")`
  3. **Assert:** 拋出 `ValidationError`，code 為 `"parameter_invalid"`，param 為 `"amount"`

#### 情境 4: 無效輸入 — 數量為負 (Invalid Input)

- **測試案例 ID:** `TC-Tx-004`
- **描述:** 數量為負數時應拒絕
- **測試步驟:**
  1. **Arrange:** 無
  2. **Act:** 呼叫 `createTransaction({ type: "income", item_name: "美式咖啡", amount: 120, quantity: -1 }, "usr_01")`
  3. **Assert:** 拋出 `ValidationError`，code 為 `"quantity_invalid"`

---

### 規格 4: `createFromQuickButton(buttonId: string, overrides: Partial, userId: string): Promise<Transaction>`

**描述:** 透過快速按鈕一鍵記錄銷售，支援覆寫杯數與售價。

**契約式設計 (DbC):**

- **前置條件 (Preconditions):**
  1. `buttonId` 必須對應一個存在的 `quick_buttons` 記錄
  2. `overrides.quantity` 若提供，必須 > 0
  3. `overrides.unit_price` 若提供，必須 > 0
- **後置條件 (Postconditions):**
  1. 交易 `item_name` 與 `category` 繼承自快速按鈕的設定
  2. 若 `overrides.quantity` 未提供，預設為 1
  3. 若 `overrides.unit_price` 未提供，使用按鈕的 `default_price`
  4. `amount` = `unit_price × quantity`
  5. 快速按鈕本身的 `default_price` **不會被修改**
- **不變性 (Invariants):**
  1. 快速按鈕的 `default_price` 在此操作前後保持不變

---

### 測試情境與案例 (Test Scenarios & Cases)

#### 情境 1: 正常路徑 — 一鍵記帳 (Happy Path)

- **測試案例 ID:** `TC-QB-Tx-001`
- **描述:** 點擊快速按鈕記錄一杯銷售
- **測試步驟:**
  1. **Arrange:** 快速按鈕 `qb_01` 存在，item_name="美式咖啡", default_price=120
  2. **Act:** 呼叫 `createFromQuickButton("qb_01", {}, "usr_01")`
  3. **Assert:**
     - `transaction.item_name` 為 `"美式咖啡"`
     - `transaction.amount` 為 `120`
     - `transaction.quantity` 為 `1`
     - `transaction.type` 為 `"income"`

#### 情境 2: 正常路徑 — 修改杯數 (Happy Path)

- **測試案例 ID:** `TC-QB-Tx-002`
- **描述:** 修改為 3 杯
- **測試步驟:**
  1. **Arrange:** 同 TC-QB-Tx-001
  2. **Act:** 呼叫 `createFromQuickButton("qb_01", { quantity: 3 }, "usr_01")`
  3. **Assert:**
     - `transaction.amount` 為 `360` (120 × 3)
     - `transaction.quantity` 為 `3`

#### 情境 3: 正常路徑 — 修改售價/促銷 (Happy Path)

- **測試案例 ID:** `TC-QB-Tx-003`
- **描述:** 促銷價 100 元 × 2 杯
- **測試步驟:**
  1. **Arrange:** 同 TC-QB-Tx-001
  2. **Act:** 呼叫 `createFromQuickButton("qb_01", { unit_price: 100, quantity: 2 }, "usr_01")`
  3. **Assert:**
     - `transaction.amount` 為 `200` (100 × 2)
     - 快速按鈕 `qb_01` 的 `default_price` 仍為 `120` (不變)

#### 情境 4: 按鈕不存在 (Sad Path)

- **測試案例 ID:** `TC-QB-Tx-004`
- **描述:** 使用不存在的按鈕 ID
- **測試步驟:**
  1. **Arrange:** `qb_999` 不存在
  2. **Act:** 呼叫 `createFromQuickButton("qb_999", {}, "usr_01")`
  3. **Assert:** 拋出 `NotFoundError`，code 為 `"resource_not_found"`

---

## 模組: AIClassifierAdapter

**對應架構文件:** `05_architecture_design.md#ai-classifier`
**對應 BDD Feature:** `epic_smart_input.feature` (US-103)

---

### 規格 5: `classifyExpense(description: string): Promise<ClassificationResult>`

**描述:** 將支出描述文字傳送至 Gemini API 進行分類，失敗時 Fallback 到本地關鍵字規則。

**契約式設計 (DbC):**

- **前置條件 (Preconditions):**
  1. `description` 不可為空
- **後置條件 (Postconditions):**
  1. 回傳 `category` 為合法分類之一 (`"進貨成本"`, `"固定支出"`, `"設備維修"`, `"未分類"`)
  2. 回傳 `source` 為 `"gemini"` 或 `"local_keyword"`
  3. 回傳 `confidence` 為 0~1 之間的浮點數
- **不變性 (Invariants):**
  1. 即使外部 API 不可用，函式仍不拋出例外 (Fallback 保底)
  2. 僅當 Gemini 與本地規則皆無法分類時，回傳 `category: "未分類"`

---

### 測試情境與案例 (Test Scenarios & Cases)

#### 情境 1: 正常路徑 — Gemini 成功分類 (Happy Path)

- **測試案例 ID:** `TC-AI-001`
- **描述:** Gemini API 正確分類「買咖啡豆 500 元」
- **測試步驟:**
  1. **Arrange:** Mock Gemini API 回傳 `{ category: "進貨成本", item_name: "咖啡豆", amount: 500 }`
  2. **Act:** 呼叫 `classifier.classifyExpense("買咖啡豆 500 元")`
  3. **Assert:**
     - `result.category` 為 `"進貨成本"`
     - `result.source` 為 `"gemini"`
     - `result.confidence` > 0.8

#### 情境 2: Fallback — Gemini 不可用 (Fallback)

- **測試案例 ID:** `TC-AI-002`
- **描述:** Gemini API Timeout，Fallback 到本地關鍵字
- **測試步驟:**
  1. **Arrange:** Mock Gemini API 拋出 `TimeoutError`
  2. **Act:** 呼叫 `classifier.classifyExpense("繳電費 800 元")`
  3. **Assert:**
     - `result.category` 為 `"固定支出"` (本地規則：「電費」→ 固定支出)
     - `result.source` 為 `"local_keyword"`

#### 情境 3: Fallback — 無法分類 (Fallback)

- **測試案例 ID:** `TC-AI-003`
- **描述:** Gemini 不可用且本地規則無法匹配
- **測試步驟:**
  1. **Arrange:** Mock Gemini API 拋出錯誤
  2. **Act:** 呼叫 `classifier.classifyExpense("雜支 100 元")`
  3. **Assert:**
     - `result.category` 為 `"未分類"`
     - `result.source` 為 `"local_keyword"`

#### 情境 4: Gemini 回傳非法分類 (Edge Case)

- **測試案例 ID:** `TC-AI-004`
- **描述:** Gemini 回傳一個不在合法列表中的分類
- **測試步驟:**
  1. **Arrange:** Mock Gemini API 回傳 `{ category: "其他亂分類" }`
  2. **Act:** 呼叫 `classifier.classifyExpense("買東西 200 元")`
  3. **Assert:**
     - `result.category` 為 `"未分類"` (不信任非法分類)
     - `result.source` 為 `"gemini"` 但已 sanitize

---

## 模組: VoiceProcessor

**對應架構文件:** `05_architecture_design.md#voice-processor`
**對應 BDD Feature:** `epic_smart_input.feature` (US-101)

---

### 規格 6: `processVoiceInput(audioBuffer: Buffer): Promise<VoiceParsedResult>`

**描述:** 將音檔傳送至 Google STT 取得文字，再解析金額/品項。

**契約式設計 (DbC):**

- **前置條件 (Preconditions):**
  1. `audioBuffer` 不可為空，大小 ≤ 10MB
  2. 格式須為 wav 或 m4a
- **後置條件 (Postconditions):**
  1. 回傳 `voice_text` 為 STT 辨識的原始文字
  2. 回傳 `parsed.amount` 為從文字中解析出的數字 (若有)
  3. 回傳 `parsed.item_name` 為從文字中解析出的品項 (若有)
- **不變性 (Invariants):**
  1. 若網路不可用，拋出 `NetworkRequiredError` 而非默默失敗

---

### 測試情境與案例 (Test Scenarios & Cases)

#### 情境 1: 正常路徑 (Happy Path)

- **測試案例 ID:** `TC-Voice-001`
- **描述:** 成功辨識「買咖啡豆 500 元」
- **測試步驟:**
  1. **Arrange:** Mock Google STT 回傳文字 `"買咖啡豆 500 元"`
  2. **Act:** 呼叫 `voiceProcessor.processVoiceInput(mockAudioBuffer)`
  3. **Assert:**
     - `result.voice_text` 為 `"買咖啡豆 500 元"`
     - `result.parsed.amount` 為 `500`
     - `result.parsed.item_name` 包含 `"咖啡豆"`

#### 情境 2: 辨識無金額 (Edge Case)

- **測試案例 ID:** `TC-Voice-002`
- **描述:** 語音中未提及金額
- **測試步驟:**
  1. **Arrange:** Mock Google STT 回傳 `"買咖啡豆"`
  2. **Act:** 呼叫 `processVoiceInput(mockAudioBuffer)`
  3. **Assert:**
     - `result.parsed.amount` 為 `null`
     - `result.voice_text` 為 `"買咖啡豆"`

#### 情境 3: 網路不可用 (Sad Path)

- **測試案例 ID:** `TC-Voice-003`
- **描述:** 無網路連線時呼叫語音辨識
- **測試步驟:**
  1. **Arrange:** Mock Google STT 拋出 `NetworkError`
  2. **Act:** 呼叫 `processVoiceInput(mockAudioBuffer)`
  3. **Assert:** 拋出 `NetworkRequiredError`，code 為 `"network_required"`

#### 情境 4: STT 辨識失敗 (Sad Path)

- **測試案例 ID:** `TC-Voice-004`
- **描述:** 噪音過大無法辨識
- **測試步驟:**
  1. **Arrange:** Mock Google STT 回傳空結果
  2. **Act:** 呼叫 `processVoiceInput(mockAudioBuffer)`
  3. **Assert:** 拋出 `VoiceRecognitionError`，code 為 `"voice_recognition_failed"`

---

## 模組: QuickButtonService

**對應架構文件:** `05_architecture_design.md#quick-button-module`
**對應 BDD Feature:** `epic_smart_input.feature` (US-105)

---

### 規格 7: `reorderButtons(order: ReorderInput[]): Promise<void>`

**描述:** 批量更新快速按鈕的顯示排序。

**契約式設計 (DbC):**

- **前置條件 (Preconditions):**
  1. `order` 陣列不可為空
  2. 每個元素的 `id` 必須對應存在的快速按鈕
  3. 每個元素的 `display_order` 必須為正整數
- **後置條件 (Postconditions):**
  1. 所有指定按鈕的 `display_order` 已更新
  2. 未在 `order` 中的按鈕不受影響
- **不變性 (Invariants):**
  1. 操作為原子性 — 全部成功或全部回滾

---

### 測試情境與案例 (Test Scenarios & Cases)

#### 情境 1: 正常路徑 (Happy Path)

- **測試案例 ID:** `TC-Reorder-001`
- **描述:** 成功重新排序 3 個按鈕
- **測試步驟:**
  1. **Arrange:** 按鈕 A(order=1), B(order=2), C(order=3) 存在
  2. **Act:** 呼叫 `reorderButtons([{ id: "C", display_order: 1 }, { id: "A", display_order: 2 }, { id: "B", display_order: 3 }])`
  3. **Assert:** 查詢後 C.display_order=1, A.display_order=2, B.display_order=3

#### 情境 2: 按鈕 ID 不存在 (Sad Path)

- **測試案例 ID:** `TC-Reorder-002`
- **描述:** 包含不存在的按鈕 ID
- **測試步驟:**
  1. **Arrange:** `qb_999` 不存在
  2. **Act:** 呼叫 `reorderButtons([{ id: "qb_999", display_order: 1 }])`
  3. **Assert:** 拋出 `NotFoundError`，且所有排序不變 (回滾)

---

## 模組: ReportGenerator

**對應架構文件:** `05_architecture_design.md#dashboard--report-module`
**對應 BDD Feature:** `epic_monthly_report.feature` (US-301, US-302, US-303) + `epic_daily_overview.feature` (US-201)

---

### 規格 8: `generateMonthlyReport(period: string): Promise<MonthlyReport>`

**描述:** 聚合指定月份的所有交易記錄，產出完整月報表。

**契約式設計 (DbC):**

- **前置條件 (Preconditions):**
  1. `period` 格式為 `YYYY-MM` (例如 `"2026-02"`)
  2. 指定月份必須已結束 (不可產生當月報表)
  3. 指定月份必須在系統啟用後 (有交易資料)
- **後置條件 (Postconditions):**
  1. `report.summary.net_profit` = `total_income - total_expense`
  2. `report.cost_breakdown` 各項 `percentage` 總和為 100%
  3. `report.top_items` 依 quantity 降序排列，最多 10 項
  4. 若有上月資料，`mom_change_percent` 正確計算
  5. 報表以 JSON 儲存至 `monthly_reports` 表
- **不變性 (Invariants):**
  1. 報表生成應在 5 秒內完成 (NFR)
  2. 同一 period 重複呼叫應覆寫而非重複建立

---

### 測試情境與案例 (Test Scenarios & Cases)

#### 情境 1: 正常路徑 (Happy Path)

- **測試案例 ID:** `TC-Report-001`
- **描述:** 成功產生 2026 年 2 月報表
- **測試步驟:**
  1. **Arrange:** 2 月有 income=85000, expense=41000 (進貨20000, 固定18000, 維修3000)；1 月 net_profit=40000
  2. **Act:** 呼叫 `reportGenerator.generateMonthlyReport("2026-02")`
  3. **Assert:**
     - `summary.net_profit` 為 `44000`
     - `summary.mom_change_percent` 為 `10.0`
     - `cost_breakdown` 有 3 項，percentage 總和為 `100`
     - `top_items` 依 quantity 降序

#### 情境 2: 首月無上月資料 (Edge Case)

- **測試案例 ID:** `TC-Report-002`
- **描述:** 系統首月 (1 月) 產出報表，無前月比較
- **測試步驟:**
  1. **Arrange:** 1 月有交易；12 月無資料
  2. **Act:** 呼叫 `generateMonthlyReport("2026-01")`
  3. **Assert:**
     - `summary.prev_month_net_profit` 為 `null`
     - `summary.mom_change_percent` 為 `null`

#### 情境 3: 嘗試產生當月報表 (Edge Case)

- **測試案例 ID:** `TC-Report-003`
- **描述:** 月中嘗試產生當月報表
- **測試步驟:**
  1. **Arrange:** 當前為 2026-02-15
  2. **Act:** 呼叫 `generateMonthlyReport("2026-02")`
  3. **Assert:** 拋出 `BusinessRuleError`，code 為 `"report_not_ready"`

---

### 規格 9: `getDailyOverview(date: Date): Promise<DailyOverview>`

**描述:** 即時計算指定日期的營收概覽與週同比。

**契約式設計 (DbC):**

- **前置條件 (Preconditions):**
  1. `date` 為合法日期
- **後置條件 (Postconditions):**
  1. `total_income` = 當日所有 type=income 的 amount 總和
  2. `total_expense` = 當日所有 type=expense 的 amount 總和
  3. `net_profit` = `total_income - total_expense`
  4. `total_cups` = 當日所有 type=income 的 quantity 總和
  5. 若上週同天有資料，`wow_comparison` 正確計算差異與百分比
- **不變性 (Invariants):**
  1. 即時查詢，不依賴快取 (Dashboard 資料永遠為最新)

---

### 測試情境與案例 (Test Scenarios & Cases)

#### 情境 1: 正常路徑 (Happy Path)

- **測試案例 ID:** `TC-Daily-001`
- **描述:** 查看有交易記錄的今日概覽
- **測試步驟:**
  1. **Arrange:** 今日有 income: 美式×10(1200) + 拿鐵×8(1200), expense: 咖啡豆(500)；上週同天 income=2000
  2. **Act:** 呼叫 `getDailyOverview(today)`
  3. **Assert:**
     - `total_income` 為 `2400`
     - `total_expense` 為 `500`
     - `net_profit` 為 `1900`
     - `total_cups` 為 `18`
     - `wow_comparison.difference` 為 `400`
     - `wow_comparison.percentage_change` 為 `20.0`

#### 情境 2: 今日無記錄 (Edge Case)

- **測試案例 ID:** `TC-Daily-002`
- **描述:** 今日無任何交易
- **測試步驟:**
  1. **Arrange:** 今日無交易
  2. **Act:** 呼叫 `getDailyOverview(today)`
  3. **Assert:**
     - `total_income` 為 `0`
     - `total_cups` 為 `0`
     - `net_profit` 為 `0`

#### 情境 3: 無上週同天資料 (Edge Case)

- **測試案例 ID:** `TC-Daily-003`
- **描述:** 系統剛啟用的第一個週三
- **測試步驟:**
  1. **Arrange:** 上週同天無資料
  2. **Act:** 呼叫 `getDailyOverview(today)`
  3. **Assert:** `wow_comparison` 為 `null`

---

**LLM Prompting Guide:**
*`「請根據以下的測試案例規格，使用 Vitest + TypeScript，為我生成一個會失敗的 TDD 單元測試。目標函式：createFromQuickButton。測試案例 ID：TC-QB-Tx-003。規格如下：[貼上測試案例文本]」`*
