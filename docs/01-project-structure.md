# 專案結構

## 概述

Postiz 建構為一個 **NX monorepo**，包含多個應用程式和共享函式庫。這種架構實現了程式碼重用、一致的工具鏈以及服務的獨立部署。

## 頂層結構

```
postiz-app/
├── apps/                 # 應用程式
│   ├── backend/         # NestJS API 伺服器
│   ├── frontend/        # Next.js 網頁應用程式
│   ├── workers/         # 背景作業處理器
│   ├── cron/            # 排程任務執行器
│   ├── commands/        # CLI 命令
│   ├── extension/       # 瀏覽器擴充功能
│   └── sdk/             # JavaScript SDK
├── libraries/           # 共享函式庫
│   ├── nestjs-libraries/    # 後端共享程式碼
│   ├── react-shared-libraries/ # 前端共享程式碼
│   └── helpers/             # 通用工具
├── .github/            # GitHub Actions 工作流程
├── Jenkins/            # Jenkins CI 設定
├── var/                # 變數資料 (docker 腳本)
└── reports/            # 測試和覆蓋率報告
```

## 應用程式 (`/apps`)

### 1. Backend (`apps/backend`)
**用途**: 主要 REST API 伺服器

**技術堆疊**:
- NestJS 框架
- Express.js
- Swagger/OpenAPI 文件
- JWT 驗證
- Throttler 速率限制

**目錄結構**:
```
apps/backend/src/
├── api/
│   └── routes/                    # API 控制器
│       ├── auth.controller.ts     # 驗證端點
│       ├── posts.controller.ts    # 貼文管理
│       ├── integrations.controller.ts # 社交整合
│       ├── analytics.controller.ts # 分析資料
│       ├── billing.controller.ts  # 付款/訂閱
│       ├── media.controller.ts    # 檔案上傳
│       ├── marketplace.controller.ts # 貼文市集
│       ├── settings.controller.ts # 使用者/組織設定
│       ├── copilot.controller.ts  # AI 助理
│       └── ...
├── public-api/
│   └── routes/                    # Public API v1
├── services/
│   └── auth/                      # 驗證服務
│       └── permissions/           # CASL 權限
├── app.module.ts                  # 根模組
└── main.ts                        # 應用程式進入點
```

**主要功能**:
- 具有 Swagger 文件的 RESTful API
- 基於角色的存取控制 (RBAC)
- 社交平台的 OAuth 整合
- Webhook 支援
- 第三方整合的 Public API

**腳本**:
- `pnpm dev:backend` - 帶熱重載的開發模式
- `pnpm build:backend` - 生產建置
- `pnpm start:prod:backend` - 啟動生產伺服器

---

### 2. Frontend (`apps/frontend`)
**用途**: Next.js 網頁應用程式

**技術堆疊**:
- Next.js 14+ (App Router)
- React 18
- TailwindCSS + Mantine UI
- TypeScript
- SWR 用於資料獲取
- Zustand 用於狀態管理

**目錄結構**:
```
apps/frontend/src/
├── app/
│   ├── (app)/                     # 已驗證路由
│   │   ├── (site)/               # 主應用程式
│   │   │   ├── launches/         # 貼文排程
│   │   │   ├── analytics/        # 分析儀表板
│   │   │   ├── marketplace/      # 貼文市集
│   │   │   ├── settings/         # 設定頁面
│   │   │   └── ...
│   │   ├── (preview)/            # 預覽頁面
│   │   ├── auth/                 # 驗證頁面
│   │   └── api/                  # API 路由
│   └── (extension)/              # 擴充功能彈出視窗
├── components/                    # React 元件
│   ├── launches/                 # 貼文排程 UI
│   ├── analytics/                # 分析圖表
│   ├── marketplace/              # 市集 UI
│   ├── settings/                 # 設定表單
│   ├── copilot/                  # AI 助理
│   └── ...
├── middleware.ts                 # Next.js 中介軟體
└── instrumentation.ts            # 監控設定
```

**主要功能**:
- 伺服器端渲染 (SSR)
- 帶中介軟體的受保護路由
- 響應式設計
- 即時更新
- 拖放檔案上傳
- 富文本編輯器 (TipTap)
- AI 驅動的內容生成

**腳本**:
- `pnpm dev:frontend` - 開發伺服器
- `pnpm build:frontend` - 生產建置
- `pnpm start:prod:frontend` - 啟動生產伺服器

---

### 3. Workers (`apps/workers`)
**用途**: 使用 BullMQ 的背景作業處理器

**技術堆疊**:
- NestJS Microservices
- BullMQ (基於 Redis 的佇列)
- 多種 worker 類型

**職責**:
- 貼文發佈到社交平台
- 媒體處理 (圖片、影片)
- Webhook 傳遞
- 電子郵件通知
- 分析資料收集
- 排程貼文執行

**主要 Workers**:
- 貼文發佈 worker
- 媒體上傳 worker
- 分析同步 worker
- 通知 worker
- 整合刷新 worker

**腳本**:
- `pnpm dev:workers` - 開發模式
- `pnpm build:workers` - 生產建置
- `pnpm start:prod:workers` - 啟動 workers

---

### 4. Cron (`apps/cron`)
**用途**: 排程任務和維護作業

**技術堆疊**:
- NestJS Schedule
- Cron 表達式

**排程任務**:
- 社交平台令牌刷新
- 分析資料彙總
- 資料庫清理
- 訂閱續訂
- 電子郵件摘要發送
- 整合健康檢查

**腳本**:
- `pnpm dev:cron` - 開發模式
- `pnpm build:cron` - 生產建置
- `pnpm start:prod:cron` - 啟動 cron 服務

---

### 5. Extension (`apps/extension`)
**用途**: 用於快速發文的瀏覽器擴充功能

**技術堆疊**:
- React
- Vite with CRXJS
- WebExtension API
- Manifest V3

**功能**:
- 從任何網頁快速建立貼文
- 內容抓取
- 圖片提取
- 與主應用程式整合

---

### 6. Commands (`apps/commands`)
**用途**: 用於資料庫操作和維護的 CLI 工具

**技術堆疊**:
- NestJS Command
- Yargs

**可用命令**:
- 資料庫種子
- 使用者管理
- 資料遷移
- 系統診斷

---

### 7. SDK (`apps/sdk`)
**用途**: Postiz API 的 JavaScript/TypeScript SDK

**功能**:
- 型別安全的 API 客戶端
- 基於 Promise 的介面
- 發布到 npm 為 `@postiz/node`

---

## 共享函式庫 (`/libraries`)

### 1. NestJS Libraries (`libraries/nestjs-libraries`)
**用途**: 後端共享程式碼和服務

**主要模組**:

```
nestjs-libraries/src/
├── database/
│   └── prisma/                    # Prisma ORM
│       ├── schema.prisma         # 資料庫模式
│       └── [entity]/             # 每個實體的服務
│           └── *.service.ts      # CRUD 操作
├── integrations/
│   └── social/                   # 社交平台整合
│       ├── twitter.provider.ts
│       ├── instagram.provider.ts
│       ├── linkedin.provider.ts
│       └── ...
├── dtos/                         # 資料傳輸物件
│   ├── auth/
│   ├── posts/
│   ├── integrations/
│   └── ...
├── agent/                        # AI 代理服務
├── bull-mq-transport-new/       # 佇列傳輸
├── emails/                       # 電子郵件模板
├── upload/                       # 檔案上傳處理
├── redis/                        # Redis 客戶端
├── openai/                       # OpenAI 整合
├── short-linking/               # URL 縮短
├── user/                        # 使用者裝飾器/守衛
└── ...
```

**主要元件**:
- **Database Services**: 基於 Prisma 的資料存取層
- **Social Integrations**: 特定平台的提供者 (20+ 平台)
- **DTOs**: 使用 class-validator 進行請求/回應驗證
- **Queues**: BullMQ 作業管理
- **Email**: 交易式電子郵件模板
- **AI**: OpenAI/LangChain 整合

---

### 2. React Shared Libraries (`libraries/react-shared-libraries`)
**用途**: 前端共享元件和工具

**模組**:
```
react-shared-libraries/src/
├── form/                         # 表單元件
├── helpers/                      # 工具函數
├── toaster/                      # Toast 通知
├── translation/                  # i18n 工具
└── sentry/                       # 錯誤追蹤
```

---

### 3. Helpers (`libraries/helpers`)
**用途**: 前端和後端的通用工具

**模組**:
```
helpers/src/
├── auth/                         # 驗證工具
├── configuration/                # 設定管理
├── decorators/                   # TypeScript 裝飾器
├── subdomain/                    # 子網域處理
├── swagger/                      # API 文件輔助工具
└── utils/                        # 通用工具
```

---

## 設定檔

### 根層級
- `package.json` - 主要套件檔案，包含工作區腳本
- `pnpm-workspace.yaml` - pnpm 工作區設定
- `tsconfig.base.json` - 基礎 TypeScript 設定
- `nx.json` - NX 工作區設定
- `docker-compose.dev.yaml` - 開發 docker 設定
- `.env.example` - 環境變數模板

### 建置與開發
- `jest.config.ts` - 測試設定
- `eslint.config.mjs` - ESLint 規則
- `.prettierrc` - 程式碼格式化

---

## 開發工作流程

### Monorepo 結構的優勢

1. **程式碼共享**: 共享函式庫防止重複
2. **一致的工具鏈**: 所有應用程式使用相同的建置工具
3. **型別安全**: 共享 TypeScript 型別
4. **原子變更**: 一起更新 API 和客戶端
5. **高效的 CI/CD**: 只建置受影響的專案

### 執行完整堆疊

```bash
# 安裝依賴
pnpm install

# 在開發模式下啟動所有服務
pnpm dev

# 啟動個別服務
pnpm dev:backend
pnpm dev:frontend
pnpm dev:workers
pnpm dev:cron

# 為生產環境建置
pnpm build
```

### 資料庫管理

```bash
# 生成 Prisma 客戶端
pnpm prisma-generate

# 推送模式變更
pnpm prisma-db-push

# 重置資料庫
pnpm prisma-reset
```

---

## 部署架構

### 生產服務

1. **Backend API** (Port 3000)
   - 處理 HTTP 請求
   - JWT 驗證
   - 速率限制

2. **Frontend** (Port 3003)
   - Next.js SSR
   - 在生產環境中透過 CDN 提供服務

3. **Workers** (背景)
   - 多個實例以實現可擴展性
   - 從 Redis 佇列處理作業

4. **Cron** (背景)
   - 用於排程任務的單一實例

5. **Database** (PostgreSQL)
   - 主要資料儲存
   - 由 Prisma 管理

6. **Redis**
   - BullMQ 作業佇列
   - 會話儲存
   - 快取

---

## 套件管理

- **套件管理器**: pnpm (版本 10.6.1)
- **Node 版本**: 22.12.0+
- **工作區**: 透過 pnpm workspaces 管理
- **依賴**: 根目錄中的共享依賴，子目錄中的應用程式特定依賴

---

## 主要設計模式

1. **服務層**: 業務邏輯與控制器分離
2. **Repository 模式**: 透過 Prisma 服務進行資料存取
3. **Factory 模式**: 動態整合載入
4. **依賴注入**: NestJS IoC 容器
5. **佇列模式**: 使用 BullMQ 進行非同步作業處理
6. **策略模式**: 不同的社交平台策略

---

## 測試結構

```
# 單元測試
*.spec.ts              # 與原始碼檔案共同放置

# E2E 測試
apps/*/test/           # 應用程式層級測試

# 測試報告
reports/               # 覆蓋率和測試結果
```

---

## 下一步

- 查看 `02-backend-api.md` 了解 API 端點詳情
- 查看 `03-frontend-architecture.md` 了解前端元件
- 查看 `04-database-models.md` 了解資料模型
- 查看 `05-api-capabilities.md` 了解 API 使用範例
