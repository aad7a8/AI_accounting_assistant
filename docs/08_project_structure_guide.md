# 專案結構指南 (Project Structure Guide) - CoffeeBook AI

---

**文件版本 (Document Version):** `v1.0`
**最後更新 (Last Updated):** `2026-02-11`
**主要作者 (Lead Author):** `[技術負責人 / Lead Engineer]`
**狀態 (Status):** `活躍 (Active)`
**對應架構文檔:** `05_architecture_design.md`
**對應 API 規範:** `06_api_design_specification.md`
**專案代號:** `CB-2026-Q1`

---

**相關文件：**
- 技術選型理由 → [架構設計文件 (05_architecture_design.md)](05_architecture_design.md) §4
- API 端點定義 → [API 設計規範 (06_api_design_specification.md)](06_api_design_specification.md)
- 模組測試規格 → [模組規格與測試 (07_module_specification_and_tests.md)](07_module_specification_and_tests.md)

## 目錄 (Table of Contents)

- [1. 指南目的 (Purpose of This Guide)](#1-指南目的-purpose-of-this-guide)
- [2. 核心設計原則 (Core Design Principles)](#2-核心設計原則-core-design-principles)
- [3. 頂層目錄結構 (Top-Level Directory Structure)](#3-頂層目錄結構-top-level-directory-structure)
- [4. 目錄詳解 (Directory Breakdown)](#4-目錄詳解-directory-breakdown)
  - [4.1 `packages/server/src/` - 後端 API Server 原始碼](#41-packagesserversrc---後端-api-server-原始碼)
  - [4.2 `apps/mobile/src/` - React Native App 原始碼](#42-appsmobilesrc---react-native-app-原始碼)
  - [4.3 `apps/web/src/` - React Web SPA 原始碼](#43-appswebsrc---react-web-spa-原始碼)
  - [4.4 `tests/` - 測試代碼](#44-tests---測試代碼)
  - [4.5 `docs/` - 文檔](#45-docs---文檔)
  - [4.6 `scripts/` - 腳本](#46-scripts---腳本)
  - [4.7 `configs/` - 環境配置](#47-configs---環境配置)
- [5. 文件命名約定 (File Naming Conventions)](#5-文件命名約定-file-naming-conventions)
- [6. 模組內部結構約定 (Module Internal Conventions)](#6-模組內部結構約定-module-internal-conventions)
- [7. 演進原則 (Evolution Principles)](#7-演進原則-evolution-principles)

---

## 1. 指南目的 (Purpose of This Guide)

*   為「家庭咖啡店智能記帳系統 (CoffeeBook AI)」提供一個標準化、可擴展且易於理解的目錄和文件結構。
*   確保團隊成員能夠快速定位代碼、配置文件和文檔，降低新成員的上手成本。
*   促進代碼的模塊化和關注點分離，提高可維護性。
*   與 `05_architecture_design.md` 中定義的 Clean Architecture 分層及 DDD 限界上下文保持一致。

## 2. 核心設計原則 (Core Design Principles)

*   **Monorepo 結構 (Monorepo Layout)：** 前端 (mobile / web) 與後端 (server) 共存於同一個 Repository，透過 `apps/` 和 `packages/` 進行區分，共享 TypeScript 型別定義與工具函式。
*   **按功能模組組織 (Organize by Feature Module)：** 後端以業務功能模組 (`auth`, `transaction`, `quick-button`, `dashboard`, `report`, `photo`, `backup`) 為目錄劃分，每個模組內自包含 Controller → Service → Repository，對應架構文件中的限界上下文 (Bounded Contexts)。
*   **明確的職責 (Clear Responsibilities)：** 每個頂層目錄都有其單一、明確的職責。模組之間僅透過公開介面 (exported functions / types) 溝通。
*   **一致的命名 (Consistent Naming)：** 目錄使用 `kebab-case`，TypeScript 文件使用 `kebab-case.ts`，與 API 路徑命名 (`/quick-buttons`, `/transactions`) 保持一致。
*   **配置外部化 (Externalized Configuration)：** 應用程式配置與代碼分離，透過 `.env` 檔案與 `configs/` 目錄管理不同環境設定。
*   **根目錄簡潔 (Clean Root Directory)：** 根目錄僅包含專案級別文件 (README, docker-compose, pyproject.toml 等)，原始碼存放在 `apps/` 與 `packages/` 下。

## 3. 頂層目錄結構 (Top-Level Directory Structure)

```plaintext
coffeebook-ai/
├── .github/                    # CI/CD 工作流程 (GitHub Actions)
│   └── workflows/
│       ├── ci.yml              # PR 檢查 (lint + test)
│       └── deploy.yml          # 部署至店內設備
├── .vscode/                    # VS Code 編輯器配置
│   ├── settings.json
│   └── extensions.json
├── apps/                       # 前端應用程式
│   ├── mobile/                 # React Native App (iOS + Android)
│   └── web/                    # React Web SPA
├── configs/                    # 環境配置文件
│   ├── .env.example            # 環境變數範本
│   ├── .env.development        # 開發環境
│   ├── .env.production         # 生產環境 (店內設備)
│   └── nginx.conf              # Nginx Reverse Proxy 設定
├── docs/                       # 專案文檔
│   ├── prd/                    # 產品需求文件
│   ├── architecture/           # 架構設計文件
│   ├── api/                    # API 規範文件
│   ├── bdd/                    # BDD 情境文件
│   ├── adrs/                   # 架構決策記錄
│   └── images/                 # 文檔用圖片
├── packages/                   # 後端服務與共享套件
│   ├── server/                 # Node.js API Server (Express)
│   └── shared/                 # (選填) 前後端共享型別定義
├── scripts/                    # 開發與運維腳本
├── .dockerignore               # Docker 忽略文件
├── .gitignore                  # Git 忽略文件
├── .pre-commit-config.yaml     # pre-commit 鉤子配置
├── docker-compose.yml          # Docker Compose 服務定義
├── LICENSE                     # 專案許可證
├── package.json                # Monorepo 根 package.json (workspaces)
├── tsconfig.base.json          # 共用 TypeScript 配置
├── turbo.json                  # (選填) Turborepo 設定
└── README.md                   # 專案介紹和快速入門指南
```

## 4. 目錄詳解 (Directory Breakdown)

### 4.1 `packages/server/src/` - 後端 API Server 原始碼

這是系統的核心，所有後端業務邏輯、API 路由、資料庫操作都在這裡。結構對應架構文件中的 7 個業務模組與 Clean Architecture 分層。

```plaintext
packages/server/
├── src/
│   ├── app.ts                          # Express 實例建立、中介層掛載、路由註冊
│   ├── server.ts                       # 啟動入口 (listen port)
│   │
│   ├── modules/                        # 業務功能模組 (按 Feature 組織)
│   │   ├── auth/                       # 🔐 身分識別上下文 (Identity Context)
│   │   │   ├── auth.controller.ts      # REST 端點 (POST /auth/login, etc.)
│   │   │   ├── auth.service.ts         # 業務邏輯 (login, createUser)
│   │   │   ├── auth.repository.ts      # 資料存取 (users CRUD)
│   │   │   ├── auth.routes.ts          # 路由定義
│   │   │   ├── auth.dto.ts             # DTOs (LoginRequest, UserCreateRequest)
│   │   │   └── auth.validator.ts       # 輸入驗證 (Zod schemas)
│   │   │
│   │   ├── transaction/                # 📒 記帳上下文 (Accounting Context)
│   │   │   ├── transaction.controller.ts
│   │   │   ├── transaction.service.ts  # createTransaction, createFromQuickButton
│   │   │   ├── transaction.repository.ts
│   │   │   ├── transaction.routes.ts
│   │   │   ├── transaction.dto.ts
│   │   │   ├── transaction.validator.ts
│   │   │   ├── voice.processor.ts      # 語音輸入處理 (Google STT 整合)
│   │   │   ├── ai-classifier.adapter.ts  # Gemini API 分類適配器
│   │   │   └── local-keyword.classifier.ts  # 本地關鍵字 Fallback 分類器
│   │   │
│   │   ├── quick-button/               # 📒 記帳上下文 (快速按鈕子模組)
│   │   │   ├── quick-button.controller.ts
│   │   │   ├── quick-button.service.ts # reorderButtons, CRUD
│   │   │   ├── quick-button.repository.ts
│   │   │   ├── quick-button.routes.ts
│   │   │   ├── quick-button.dto.ts
│   │   │   └── quick-button.validator.ts
│   │   │
│   │   ├── dashboard/                  # 📊 報表上下文 (Reporting Context) — 即時查詢
│   │   │   ├── dashboard.controller.ts
│   │   │   ├── dashboard.service.ts    # getDailyOverview, getTrend, getRanking
│   │   │   ├── dashboard.repository.ts
│   │   │   └── dashboard.routes.ts
│   │   │
│   │   ├── report/                     # 📊 報表上下文 — 月報表快照
│   │   │   ├── report.controller.ts
│   │   │   ├── report.service.ts       # generateMonthlyReport
│   │   │   ├── report.repository.ts
│   │   │   └── report.routes.ts
│   │   │
│   │   ├── photo/                      # 📒 記帳上下文 (照片附件子模組)
│   │   │   ├── photo.controller.ts
│   │   │   ├── photo.service.ts
│   │   │   └── photo.routes.ts
│   │   │
│   │   └── backup/                     # 🌐 外部整合上下文 (Integration Context)
│   │       ├── backup.scheduler.ts     # Cron Job (每日 03:00)
│   │       └── google-drive.adapter.ts # Google Drive 備份適配器
│   │
│   ├── shared/                         # 跨模組共用元件
│   │   ├── middleware/                 # Express 中介層
│   │   │   ├── auth.middleware.ts      # JWT 驗證
│   │   │   ├── rbac.middleware.ts      # 角色權限檢查
│   │   │   ├── rate-limit.middleware.ts # 速率限制
│   │   │   ├── error-handler.middleware.ts # 全域錯誤處理
│   │   │   └── request-id.middleware.ts   # X-Request-ID 追蹤
│   │   ├── domain/                     # 共用 Domain 物件
│   │   │   ├── value-objects/          # Value Objects (Money, Category)
│   │   │   │   ├── money.vo.ts
│   │   │   │   └── category.vo.ts
│   │   │   └── errors/                 # 自定義領域例外
│   │   │       ├── base.error.ts
│   │   │       ├── authentication.error.ts
│   │   │       ├── validation.error.ts
│   │   │       ├── not-found.error.ts
│   │   │       ├── conflict.error.ts
│   │   │       ├── business-rule.error.ts
│   │   │       └── network-required.error.ts
│   │   ├── types/                      # 共用 TypeScript 型別
│   │   │   ├── express.d.ts            # Express Request 擴展 (req.user)
│   │   │   └── api-response.type.ts    # 標準回應封裝型別
│   │   └── utils/                      # 工具函式
│   │       ├── jwt.util.ts             # JWT 簽發與驗證
│   │       ├── hash.util.ts            # bcrypt 雜湊
│   │       └── date.util.ts            # 日期處理 (ISO 8601)
│   │
│   └── prisma/                         # 資料庫 ORM
│       ├── schema.prisma               # Prisma Schema (資料模型定義)
│       ├── migrations/                 # 資料庫遷移記錄
│       └── seed.ts                     # 種子資料 (初始 Owner 帳號)
│
├── package.json
├── tsconfig.json
├── vitest.config.ts                    # Vitest 測試框架配置
└── nodemon.json                        # 開發環境熱重載
```

**模組與限界上下文對照：**

| 限界上下文 | 對應模組目錄 | 核心職責 |
| :--- | :--- | :--- |
| 🔐 身分識別 (Identity) | `modules/auth/` | 登入、JWT 簽發、RBAC、使用者 CRUD |
| 📒 記帳 (Accounting) | `modules/transaction/` + `quick-button/` + `photo/` | 交易 CRUD、語音輸入、AI 分類、快速按鈕、照片附件 |
| 📊 報表 (Reporting) | `modules/dashboard/` + `report/` | 每日概覽、趨勢圖、排行榜、月報表生成 |
| 🌐 外部整合 (Integration) | `modules/backup/` + 各模組內 `*.adapter.ts` | Google Drive 備份、STT、Gemini 分類 |

---

### 4.2 `apps/mobile/src/` - React Native App 原始碼

```plaintext
apps/mobile/
├── src/
│   ├── screens/                    # 畫面元件 (按功能劃分)
│   │   ├── auth/                   # 登入畫面
│   │   │   └── LoginScreen.tsx
│   │   ├── home/                   # 主記帳畫面 (快速按鈕 + 語音)
│   │   │   └── HomeScreen.tsx
│   │   ├── dashboard/              # 每日概覽儀表板
│   │   │   └── DashboardScreen.tsx
│   │   ├── report/                 # 月報表查看
│   │   │   └── ReportScreen.tsx
│   │   └── settings/               # 設定 (快速按鈕管理、帳號管理)
│   │       ├── QuickButtonManageScreen.tsx
│   │       └── UserManageScreen.tsx
│   ├── components/                 # 共用 UI 元件
│   │   ├── QuickButton.tsx
│   │   ├── TransactionCard.tsx
│   │   ├── TrendChart.tsx
│   │   └── PieChart.tsx
│   ├── services/                   # API 呼叫層
│   │   ├── api-client.ts           # Axios 實例 (Base URL, JWT interceptor)
│   │   ├── auth.service.ts
│   │   ├── transaction.service.ts
│   │   ├── dashboard.service.ts
│   │   └── report.service.ts
│   ├── stores/                     # 狀態管理 (Zustand / React Context)
│   │   ├── auth.store.ts
│   │   └── transaction.store.ts
│   ├── navigation/                 # React Navigation 路由設定
│   │   └── AppNavigator.tsx
│   ├── hooks/                      # 自定義 Hooks
│   │   ├── useVoiceInput.ts
│   │   └── useOfflineSync.ts
│   ├── utils/                      # 工具函式
│   │   └── format.ts               # 金額格式化、日期格式化
│   └── App.tsx                     # 應用程式根元件
├── ios/                            # iOS 原生配置
├── android/                        # Android 原生配置
├── package.json
└── tsconfig.json
```

---

### 4.3 `apps/web/src/` - React Web SPA 原始碼

```plaintext
apps/web/
├── src/
│   ├── pages/                      # 頁面元件 (React Router)
│   │   ├── LoginPage.tsx
│   │   ├── DashboardPage.tsx
│   │   ├── TransactionsPage.tsx
│   │   ├── ReportPage.tsx
│   │   └── SettingsPage.tsx
│   ├── components/                 # 共用 UI 元件 (與 mobile 結構對齊)
│   ├── services/                   # API 呼叫層 (與 mobile 邏輯一致)
│   ├── stores/                     # 狀態管理
│   ├── hooks/                      # 自定義 Hooks
│   ├── styles/                     # 全域樣式 / Tailwind 配置
│   └── App.tsx
├── public/
│   └── index.html
├── package.json
├── tsconfig.json
└── vite.config.ts                  # Vite 建置工具配置
```

---

### 4.4 `tests/` - 測試代碼

測試代碼結構與 `packages/server/src/modules/` 保持鏡像對應，便於定位被測試的模組。測試框架使用 **Vitest + TypeScript**，對應 `07_module_specification_and_tests.md` 中定義的測試案例。

```plaintext
packages/server/
├── tests/
│   ├── setup.ts                    # 全域測試設定 (DB 連線、mock 初始化)
│   ├── helpers/                    # 測試輔助工具
│   │   ├── test-db.ts              # 測試用 DB 初始化/清除
│   │   └── factories.ts            # 測試數據工廠 (User, Transaction, QuickButton)
│   │
│   ├── unit/                       # 單元測試 (隔離外部依賴)
│   │   ├── auth/
│   │   │   ├── auth.service.test.ts       # TC-Auth-001 ~ TC-Auth-004
│   │   │   └── auth.service.create.test.ts # TC-User-001 ~ TC-User-004
│   │   ├── transaction/
│   │   │   ├── transaction.service.test.ts # TC-Tx-001 ~ TC-Tx-004
│   │   │   ├── quick-button-tx.test.ts     # TC-QB-Tx-001 ~ TC-QB-Tx-003
│   │   │   ├── ai-classifier.test.ts       # TC-AI-001 ~ TC-AI-004
│   │   │   └── voice.processor.test.ts     # TC-Voice-001 ~ TC-Voice-004
│   │   ├── quick-button/
│   │   │   └── reorder.test.ts             # TC-Reorder-001 ~ TC-Reorder-002
│   │   └── report/
│   │       ├── monthly-report.test.ts      # TC-Report-001 ~ TC-Report-003
│   │       └── daily-overview.test.ts      # TC-Daily-001 ~ TC-Daily-003
│   │
│   ├── integration/                # 整合測試 (含真實 DB 操作)
│   │   ├── auth.integration.test.ts
│   │   ├── transaction.integration.test.ts
│   │   └── report.integration.test.ts
│   │
│   └── e2e/                        # 端對端測試 (API 層級)
│       ├── auth.e2e.test.ts        # 登入 → Token → 呼叫 API 完整流程
│       ├── transaction.e2e.test.ts
│       └── dashboard.e2e.test.ts
```

**測試案例 ID 與文件對照：**

| 模組規格文件 ID | 測試文件 | 說明 |
| :--- | :--- | :--- |
| TC-Auth-001 ~ 004 | `unit/auth/auth.service.test.ts` | AuthService.login |
| TC-User-001 ~ 004 | `unit/auth/auth.service.create.test.ts` | AuthService.createUser |
| TC-Tx-001 ~ 004 | `unit/transaction/transaction.service.test.ts` | TransactionService.createTransaction |
| TC-QB-Tx-001 ~ 003 | `unit/transaction/quick-button-tx.test.ts` | TransactionService.createFromQuickButton |
| TC-AI-001 ~ 004 | `unit/transaction/ai-classifier.test.ts` | AIClassifierAdapter.classifyExpense |
| TC-Voice-001 ~ 004 | `unit/transaction/voice.processor.test.ts` | VoiceProcessor.processVoiceInput |
| TC-Reorder-001 ~ 002 | `unit/quick-button/reorder.test.ts` | QuickButtonService.reorderButtons |
| TC-Report-001 ~ 003 | `unit/report/monthly-report.test.ts` | ReportGenerator.generateMonthlyReport |
| TC-Daily-001 ~ 003 | `unit/report/daily-overview.test.ts` | ReportGenerator.getDailyOverview |

---

### 4.5 `docs/` - 文檔

所有與專案相關的長篇文檔都存放在此，版本控制隨 Git 一同管理。

```plaintext
docs/
├── prd/
│   └── 02_prd.md       # 產品需求文件 (v1.5)
├── architecture/
│   ├── 05_architecture_design.md   # 整合性架構與設計文件
│   └── 08_project_structure_guide.md # 本文件
├── api/
│   └── 06_api_design_specification.md  # API 設計規範
├── bdd/
│   └── 03_bdd_scenarios.md         # BDD 行為驅動情境
├── modules/
│   └── 07_module_specification_and_tests.md # 模組規格與測試案例
├── adrs/                                   # 架構決策記錄
│   ├── adr-001-local-first-architecture.md
│   ├── adr-002-postgresql-over-sqlite.md
│   └── adr-003-gemini-with-local-fallback.md
└── images/                                 # 文檔用圖片
    ├── c4-context-diagram.png
    ├── c4-container-diagram.png
    └── er-diagram.png
```

---

### 4.6 `scripts/` - 腳本

存放用於自動化開發、部署或維護任務的腳本。

```plaintext
scripts/
├── dev-setup.sh                # 首次開發環境設置 (安裝依賴、DB migrate、seed)
├── lint.sh                     # ESLint + Prettier 檢查
├── test.sh                     # 執行全部測試 (unit + integration)
├── test-unit.sh                # 僅執行單元測試
├── build.sh                    # 建置生產環境
├── deploy-local.sh             # 部署至店內設備 (Docker Compose)
├── db-migrate.sh               # 資料庫遷移
├── db-seed.sh                  # 種子資料填充 (初始 Owner)
├── backup-manual.sh            # 手動觸發備份
└── generate-monthly-report.sh  # 手動觸發月報表生成
```

---

### 4.7 `configs/` - 環境配置

```plaintext
configs/
├── .env.example                # 環境變數範本 (含說明)
├── .env.development            # 開發環境 (localhost:3000)
├── .env.production             # 生產環境 (店內設備 IP)
├── .env.test                   # 測試環境 (測試用 DB)
└── nginx.conf                  # Nginx Reverse Proxy 配置 (HTTPS 選配)
```

**`.env.example` 關鍵變數：**

```bash
# Server
PORT=3000
NODE_ENV=development

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/coffeebook

# JWT
JWT_SECRET=your-secret-key
JWT_EXPIRES_IN=24h

# External APIs (非核心依賴，無設定時使用 Fallback)
GOOGLE_STT_API_KEY=
GEMINI_API_KEY=
GOOGLE_DRIVE_CREDENTIALS_PATH=

# Backup
BACKUP_CRON_SCHEDULE=0 3 * * *
BACKUP_ENCRYPTION_KEY=

# Rate Limiting
LOGIN_RATE_LIMIT=5
GENERAL_RATE_LIMIT=200
```

---

## 5. 文件命名約定 (File Naming Conventions)

| 類型 | 約定 | 範例 |
| :--- | :--- | :--- |
| **目錄** | `kebab-case` | `quick-button/`, `shared/` |
| **TypeScript 模組** | `kebab-case.ts` | `auth.service.ts`, `ai-classifier.adapter.ts` |
| **TypeScript 型別定義** | `kebab-case.type.ts` / `.dto.ts` / `.vo.ts` | `api-response.type.ts`, `money.vo.ts` |
| **測試文件** | `*.test.ts` (Vitest 慣例) | `auth.service.test.ts` |
| **React 元件** | `PascalCase.tsx` | `QuickButton.tsx`, `LoginScreen.tsx` |
| **Markdown 文件** | `kebab-case.md` | `coffeebook-architecture-design.md` |
| **Shell 腳本** | `kebab-case.sh` | `dev-setup.sh`, `deploy-local.sh` |
| **環境配置** | `.env.{environment}` | `.env.development`, `.env.production` |

**模組內文件後綴約定 (後端)：**

| 後綴 | 職責 | 對應 Clean Architecture 層 |
| :--- | :--- | :--- |
| `.controller.ts` | REST 端點處理 (req → res) | Presentation Layer |
| `.routes.ts` | Express 路由定義 | Presentation Layer |
| `.service.ts` | 業務邏輯 / Use Case | Application Layer |
| `.repository.ts` | 資料庫存取 (Prisma) | Infrastructure Layer |
| `.dto.ts` | 資料傳輸對象 | Application Layer |
| `.validator.ts` | 輸入驗證 (Zod Schema) | Application Layer |
| `.adapter.ts` | 外部 API 適配器 | Infrastructure Layer (ACL) |
| `.vo.ts` | Value Object | Domain Layer |
| `.error.ts` | 領域例外定義 | Domain Layer |
| `.middleware.ts` | Express 中介層 | Infrastructure Layer |

---

## 6. 模組內部結構約定 (Module Internal Conventions)

每個功能模組遵循統一的內部結構與依賴方向：

```
┌──────────────────────────────────────────┐
│             *.routes.ts                   │  ← 路由定義
│                  ↓                        │
│           *.controller.ts                 │  ← 請求/回應處理
│                  ↓                        │
│    *.validator.ts → *.service.ts          │  ← 驗證 + 業務邏輯
│                        ↓                  │
│              *.repository.ts              │  ← 資料庫操作
│              *.adapter.ts                 │  ← 外部 API 整合
└──────────────────────────────────────────┘
```

**依賴規則：**
- `controller` 依賴 `service`，不直接操作 `repository`
- `service` 依賴 `repository` 介面，不依賴具體 ORM 實現
- `adapter` 將外部 API 回傳格式轉換為內部 Domain 物件 (防腐層)
- `shared/domain/errors/` 中的錯誤類別可被任何模組使用

---

## 7. 演進原則 (Evolution Principles)

*   **Phase 1 (MVP)：** 優先建立 `auth/`, `transaction/`, `quick-button/`, `dashboard/` 四個模組，對應架構文件 §10.1 定義的 MVP 範圍。
*   **Phase 2 (完整版)：** 新增 `report/`, `photo/`, `backup/` 模組，以及 `apps/web/` 前端。
*   **Phase 3 (增強版)：** 若功能需求顯著增長 (如庫存管理、支付整合)，可在 `modules/` 下新增對應目錄 (`inventory/`, `payment/`)，無需重構現有結構。
*   任何對頂層目錄結構的重大變更，都應透過 ADR (`docs/adrs/`) 進行記錄。
*   **保持結構的清晰和一致性比嚴格遵守某個特定模式更重要。**

---

**文件審核記錄 (Review History):**

| 日期 | 審核人 | 版本 | 變更摘要 |
| :--- | :--- | :--- | :--- |
| 2026-02-11 | [技術負責人] | v1.0 | 初稿完成，涵蓋 Monorepo 全結構、模組對照、命名約定與演進策略 |
