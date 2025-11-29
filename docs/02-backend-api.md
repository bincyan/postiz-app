# Backend API 架構

## 概述

Postiz 後端使用 **NestJS** 建構，這是一個漸進式的 Node.js 框架。它為網頁應用程式提供私有驗證 API，並為第三方整合提供公開 API。

## 架構

### 技術堆疊
- **框架**: NestJS 10.x
- **執行環境**: Node.js 22.12.0+
- **資料庫**: PostgreSQL with Prisma ORM
- **佇列**: Redis + BullMQ
- **驗證**: JWT (JSON Web Tokens)
- **API 文件**: Swagger/OpenAPI
- **速率限制**: @nestjs/throttler
- **監控**: Sentry

### 應用程式結構

```
apps/backend/src/
├── api/
│   └── routes/           # 主要 API 控制器
├── public-api/
│   └── routes/           # Public API v1
├── services/
│   └── auth/             # 驗證服務
│       └── permissions/  # 基於 CASL 的授權
├── app.module.ts         # 根應用程式模組
└── main.ts              # 啟動與設定
```

---

## API 控制器

後端公開了 22 個主要控制器，每個處理特定領域:

### 1. 驗證與使用者

#### `auth.controller.ts`
**端點**: `/auth/*`

**功能**:
- 使用者註冊和登入
- OAuth 回呼處理
- 密碼重置
- 電子郵件驗證
- 會話管理
- 令牌刷新

**主要端點**:
- `POST /auth/register` - 使用者註冊
- `POST /auth/login` - 使用者登入
- `POST /auth/forgot-password` - 請求密碼重置
- `POST /auth/reset-password` - 使用令牌重置密碼
- `GET /auth/callback/:provider` - OAuth 回呼
- `POST /auth/refresh` - 刷新 JWT 令牌

#### `users.controller.ts`
**端點**: `/users/*`

**功能**:
- 使用者個人資料管理
- 使用者偏好設定
- 頭像上傳
- 時區設定

---

### 2. 社交媒體整合

#### `integrations.controller.ts`
**端點**: `/integrations/*`

**功能**:
- 連接社交媒體帳號
- 管理已連接的整合
- OAuth 流程啟動
- 令牌刷新
- 整合設定
- 帳號分組
- 自訂客戶名稱

**主要端點**:
- `GET /integrations` - 列出所有可用平台
- `POST /integrations/connect` - 啟動 OAuth 連接
- `GET /integrations/:id` - 取得整合詳情
- `PUT /integrations/:id/group` - 更新整合群組
- `DELETE /integrations/:id` - 斷開整合
- `POST /integrations/:id/refresh` - 刷新存取令牌
- `GET /integrations/customers` - 取得客戶清單

**支援的平台**:
- X (Twitter)
- Instagram
- LinkedIn
- Facebook
- TikTok
- YouTube
- Pinterest
- Reddit
- Discord
- Slack
- Mastodon
- Bluesky
- Threads
- 以及更多...

---

### 3. 貼文管理

#### `posts.controller.ts`
**端點**: `/posts/*`

**功能**:
- 建立和排程貼文
- 編輯草稿貼文
- 刪除貼文
- 貼文評論
- 標籤管理
- AI 內容生成
- 貼文統計
- 市集整合

**主要端點**:
- `GET /posts` - 列出貼文 (帶篩選器)
- `POST /posts` - 建立新貼文
- `GET /posts/:id` - 取得貼文詳情
- `PUT /posts/:id` - 更新貼文
- `DELETE /posts/:id` - 刪除貼文
- `POST /posts/:id/comments` - 新增評論
- `GET /posts/:id/statistics` - 取得貼文分析
- `GET /posts/tags` - 列出標籤
- `POST /posts/tags` - 建立標籤
- `POST /posts/generate` - AI 驅動的貼文生成

**貼文排程**:
- 立即發布
- 排程未來貼文
- 循環貼文
- 草稿管理

---

### 4. 媒體管理

#### `media.controller.ts`
**端點**: `/media/*`

**功能**:
- 檔案上傳 (圖片、影片、PDF)
- 媒體庫管理
- 圖片處理
- 影片轉碼
- 檔案儲存 (S3 或本地)

**主要端點**:
- `POST /media/upload` - 上傳檔案
- `GET /media` - 列出媒體檔案
- `DELETE /media/:id` - 刪除媒體
- `GET /media/:id` - 取得媒體 URL

**支援的格式**:
- 圖片: PNG, JPG, GIF, WebP
- 影片: MP4, MOV, AVI
- 文件: PDF
- 最大檔案大小: 可設定

---

### 5. 分析

#### `analytics.controller.ts`
**端點**: `/analytics/*`

**功能**:
- 社交媒體指標
- 互動分析
- 追蹤者成長
- 貼文表現
- 平台特定見解

**主要端點**:
- `GET /analytics/overview` - 整體統計
- `GET /analytics/integration/:id` - 平台特定分析
- `GET /analytics/posts` - 貼文表現指標
- `GET /analytics/engagement` - 互動趨勢

**提供的指標**:
- 曝光次數
- 按讚數
- 評論數
- 分享數
- 點擊數
- 追蹤者成長
- 互動率

---

### 6. AI 與自動化

#### `copilot.controller.ts`
**端點**: `/copilot/*`

**功能**:
- AI 內容生成
- 貼文建議
- Hashtag 推薦
- 內容最佳化
- 圖片生成

**AI 功能**:
- OpenAI GPT 整合
- LangChain 工作流程
- 自訂 AI 代理
- 內容個人化

#### `autopost.controller.ts`
**端點**: `/autopost/*`

**功能**:
- RSS feed 監控
- 自動發文規則
- 內容策展
- 排程自動化

---

### 7. 市集

#### `marketplace.controller.ts`
**端點**: `/marketplace/*`

**功能**:
- 買賣社交媒體貼文
- 瀏覽可用貼文
- 貼文競價
- 交易管理

**主要端點**:
- `GET /marketplace/posts` - 瀏覽市集
- `POST /marketplace/offer` - 出價
- `PUT /marketplace/accept/:id` - 接受出價
- `GET /marketplace/purchases` - 查看購買記錄

#### `messages.controller.ts`
**端點**: `/messages/*`

**功能**:
- 市集訊息
- 買賣雙方溝通
- 訊息串

---

### 8. 帳單與訂閱

#### `billing.controller.ts`
**端點**: `/billing/*`

**功能**:
- 訂閱管理
- 方案升級/降級
- 使用追蹤
- 付款歷史

**主要端點**:
- `GET /billing/subscription` - 目前訂閱
- `POST /billing/upgrade` - 升級方案
- `POST /billing/cancel` - 取消訂閱
- `GET /billing/invoices` - 付款歷史

#### `stripe.controller.ts`
**端點**: `/stripe/*`

**功能**:
- Stripe webhook 處理
- 付款處理
- 訂閱事件

---

### 9. 組織管理

#### `agencies.controller.ts`
**端點**: `/agencies/*`

**功能**:
- 多組織支援
- 團隊管理
- 組織設定
- 成員邀請

**主要端點**:
- `GET /agencies` - 列出組織
- `POST /agencies` - 建立組織
- `POST /agencies/:id/invite` - 邀請成員
- `PUT /agencies/:id/settings` - 更新設定

---

### 10. 設定與配置

#### `settings.controller.ts`
**端點**: `/settings/*`

**功能**:
- 使用者偏好設定
- 組織設定
- 通知偏好設定
- API 金鑰管理

#### `notifications.controller.ts`
**端點**: `/notifications/*`

**功能**:
- 應用程式內通知
- 電子郵件通知
- 通知偏好設定
- 標記為已讀

---

### 11. 進階功能

#### `sets.controller.ts`
**端點**: `/sets/*`

**功能**:
- 預定義發文排程
- 內容模板
- 批次操作

#### `signature.controller.ts`
**端點**: `/signature/*`

**功能**:
- 貼文簽名
- 品牌元素
- 浮水印

#### `webhooks.controller.ts`
**端點**: `/webhooks/*`

**功能**:
- Webhook 配置
- 事件訂閱
- Webhook 傳遞日誌

#### `third-party.controller.ts`
**端點**: `/third-party/*`

**功能**:
- 第三方整合
- 外部服務連接
- API 橋接

---

## Public API (`/api/v1/public`)

### 用途
使第三方開發人員能夠以程式方式與 Postiz 整合。

### 驗證
使用 API 金鑰進行驗證:
```
Authorization: Bearer YOUR_API_KEY
```

### 可用端點

#### Public Integrations Controller
**基礎路徑**: `/api/v1/public/integrations`

**端點**:
- `GET /api/v1/public/integrations` - 列出已連接的整合
- `POST /api/v1/public/integrations/post` - 建立和排程貼文
- `GET /api/v1/public/integrations/:id/posts` - 取得整合的貼文

### 速率限制
- 預設: 每個 API 金鑰每分鐘 100 次請求
- 可依組織層級設定

---

## 驗證與授權

### JWT 驗證

**令牌結構**:
```json
{
  "userId": "uuid",
  "orgId": "uuid",
  "email": "user@example.com",
  "iat": 1234567890,
  "exp": 1234567890
}
```

**令牌生命週期**:
- 存取令牌: 1 小時
- 刷新令牌: 30 天

### 授權 (CASL)

**權限系統**:
- 基於資源的權限
- 基於動作的控制 (read, create, update, delete)
- 組織範圍的存取

**角色**:
- Owner: 完整存取
- Admin: 管理成員和內容
- User: 建立和管理自己的貼文
- Viewer: 唯讀存取

---

## 資料驗證

### DTOs (資料傳輸物件)

位於 `libraries/nestjs-libraries/src/dtos/`

**驗證堆疊**:
- `class-validator` - 基於裝飾器的驗證
- `class-transformer` - 型別轉換
- 複雜規則的自訂驗證器

**範例 DTO**:
```typescript
export class CreatePostDto {
  @IsString()
  @IsNotEmpty()
  content: string;

  @IsArray()
  @ValidateNested({ each: true })
  integrations: IntegrationDto[];

  @IsDateString()
  @IsOptional()
  scheduledAt?: string;

  @IsArray()
  @IsOptional()
  media?: string[];
}
```

---

## 佇列系統 (BullMQ)

### 作業類型

1. **貼文發布**
   - 排程貼文發布
   - 失敗重試邏輯
   - 平台特定格式化

2. **媒體處理**
   - 圖片最佳化
   - 影片轉碼
   - 縮圖生成

3. **分析同步**
   - 定期指標獲取
   - 資料彙總

4. **通知**
   - 電子郵件發送
   - 應用程式內通知

### 佇列配置

```typescript
{
  defaultJobOptions: {
    attempts: 3,
    backoff: {
      type: 'exponential',
      delay: 2000
    }
  }
}
```

---

## 錯誤處理

### HTTP 狀態碼

- `200` - 成功
- `201` - 已建立
- `400` - 錯誤請求 (驗證錯誤)
- `401` - 未授權 (遺失/無效令牌)
- `403` - 禁止 (權限不足)
- `404` - 未找到
- `429` - 請求過多 (速率限制)
- `500` - 內部伺服器錯誤

### 錯誤回應格式

```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Invalid email format"
    }
  ]
}
```

---

## 資料庫存取層

### Prisma Services

每個實體都有專用服務:

```
libraries/nestjs-libraries/src/database/prisma/
├── posts/
│   └── posts.service.ts
├── integrations/
│   └── integration.service.ts
├── users/
│   └── users.service.ts
└── ...
```

**服務模式**:
```typescript
@Injectable()
export class PostsService {
  constructor(private prisma: DatabaseService) {}

  async createPost(data: CreatePostDto) {
    return this.prisma.post.create({ data });
  }

  async getPosts(orgId: string, filters: GetPostsDto) {
    return this.prisma.post.findMany({
      where: { orgId, ...filters },
      include: { integration: true, media: true }
    });
  }
}
```

---

## 社交平台整合

### Integration Manager

位置: `libraries/nestjs-libraries/src/integrations/`

**提供者模式**:
每個社交平台都有一個提供者類別實作:
- OAuth 流程
- 貼文發布
- 媒體上傳
- 分析獲取
- 個人資料資訊

**範例平台**:
- `twitter.provider.ts` - X/Twitter
- `instagram.provider.ts` - Instagram
- `linkedin.provider.ts` - LinkedIn
- `facebook.provider.ts` - Facebook
- `tiktok.provider.ts` - TikTok
- ... 以及 15+ 個平台

### 提供者介面

```typescript
interface SocialProvider {
  authenticate(code: string): Promise<AuthToken>;
  publishPost(post: Post): Promise<PublishResult>;
  uploadMedia(file: Buffer): Promise<MediaId>;
  getAnalytics(period: DateRange): Promise<Analytics>;
  refreshToken(token: RefreshToken): Promise<AuthToken>;
}
```

---

## API 文件 (Swagger)

### 存取
- 開發: `http://localhost:3000/api`
- 生產: `https://api.postiz.com/api`

### 功能
- 互動式 API 瀏覽器
- 請求/回應模式
- 驗證測試
- 程式碼生成

---

## 效能最佳化

### 快取策略
- 對經常存取的資料使用 Redis 快取
- 更新時快取失效
- 基於 TTL 的過期

### 資料庫最佳化
- 連接池
- 使用索引進行查詢最佳化
- 急切載入關聯
- 大型資料集的分頁

### 速率限制
- 每個端點限制
- 基於使用者/組織的配額
- 分散式速率限制 (Redis)

---

## 監控與日誌

### Sentry 整合
- 錯誤追蹤
- 效能監控
- 發布追蹤

### 日誌
- 結構化日誌 (JSON)
- 日誌等級: debug, info, warn, error
- 請求/回應日誌
- 資料庫查詢日誌 (開發)

---

## 環境配置

### 必需變數

```env
# 資料庫
DATABASE_URL=postgresql://user:password@localhost:5432/postiz

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-secret-key

# OAuth 憑證 (每個平台)
TWITTER_CLIENT_ID=...
TWITTER_CLIENT_SECRET=...
INSTAGRAM_CLIENT_ID=...
# ... 等

# AWS S3 (可選)
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
S3_BUCKET=...

# 電子郵件 (Resend)
RESEND_API_KEY=...

# 監控
SENTRY_DSN=...
```

---

## API 最佳實踐

### 請求格式
- 請求主體使用 JSON
- 包含 `Content-Type: application/json` 標頭
- 篩選器使用查詢參數

### 回應格式
- 一致的 JSON 結構
- ISO 8601 日期
- 分頁元資料

### 安全性
- 生產環境始終使用 HTTPS
- 驗證所有輸入
- 清理使用者內容
- 對所有端點進行速率限制
- 使用參數化查詢 (Prisma)

---

## 下一步

- 查看 `03-frontend-architecture.md` 了解前端整合
- 查看 `04-database-models.md` 了解資料結構
- 查看 `05-api-capabilities.md` 了解使用範例
