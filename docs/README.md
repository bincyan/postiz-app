# Postiz 應用程式文件

本目錄包含 Postiz 應用程式的完整文件，這是一個開源的 AI 社群媒體排程工具。

## 文件概覽

本文件套件提供以下詳細資訊：

1. **專案結構** - 程式碼庫的架構和組織
2. **後端 API** - REST API 端點、控制器和服務
3. **前端架構** - Next.js 應用程式結構和元件
4. **資料庫模型** - Prisma 架構和資料關係
5. **API 功能** - 您可以使用 Postiz API 做什麼

## 什麼是 Postiz？

Postiz 是一個全方位的社群媒體管理平台，能夠：
- 跨多個社群媒體平台排程貼文
- AI 驅動的內容生成
- 分析和效能追蹤
- 團隊協作功能
- 買賣貼文的市集
- API 自動化支援

## 支援的平台

- X (Twitter)
- Instagram
- YouTube
- LinkedIn
- Reddit
- TikTok
- Facebook
- Pinterest
- Threads
- Discord
- Slack
- Mastodon
- Bluesky
- 更多...

## 文件檔案

- `01-project-structure.md` - 單一儲存庫結構的詳細分解
- `02-backend-api.md` - 後端 API 架構和端點
- `03-frontend-architecture.md` - 前端應用程式結構
- `04-database-models.md` - 資料庫架構和關係
- `05-api-capabilities.md` - 完整的 API 使用指南

## 技術堆疊

### 後端
- **框架**: NestJS (Node.js)
- **資料庫**: PostgreSQL 搭配 Prisma ORM
- **佇列**: Redis 搭配 BullMQ
- **API 文件**: Swagger/OpenAPI
- **身份驗證**: 基於 JWT 的身份驗證
- **電子郵件**: Resend 用於通知

### 前端
- **框架**: Next.js 14+ (React 18)
- **UI 函式庫**: Mantine、TailwindCSS
- **狀態管理**: Zustand、SWR
- **表單**: React Hook Form 搭配 Yup 驗證
- **富文本**: TipTap 編輯器
- **檔案上傳**: Uppy

### 基礎設施
- **單一儲存庫**: NX 工作區
- **套件管理器**: pnpm
- **Node 版本**: 22.12.0
- **TypeScript**: 5.5.4

## 快速導覽

- 用於 API 整合 → 參見 `05-api-capabilities.md`
- 用於資料庫查詢 → 參見 `04-database-models.md`
- 用於後端開發 → 參見 `02-backend-api.md`
- 用於前端開發 → 參見 `03-frontend-architecture.md`
- 用於專案概覽 → 參見 `01-project-structure.md`

## 儲存庫資訊

- **授權**: AGPL-3.0
- **GitHub**: https://github.com/gitroomhq/postiz-app
- **文件**: https://docs.postiz.com
- **公開 API**: https://docs.postiz.com/public-api

## 開始使用

請參閱主儲存庫 README 以取得安裝和設定說明：
https://docs.postiz.com/quickstart
