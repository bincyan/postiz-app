# 資料庫模型

## 概述

Postiz 使用 **PostgreSQL** 作為主要資料庫，並使用 **Prisma** 作為 ORM。模式結構良好，具有適當的關聯、索引和約束。

## 資料庫配置

**ORM**: Prisma 6.5.0
**資料庫**: PostgreSQL
**模式位置**: `libraries/nestjs-libraries/src/database/prisma/schema.prisma`
**遷移**: 透過 `prisma db push` 管理

---

## 核心模型

### 1. User

**用途**: 代表平台的個別使用者

**欄位**:
```prisma
model User {
  id                    String       @id @default(uuid())
  email                 String
  password              String?      # OAuth 使用者為 Null
  providerName          Provider     # LOCAL, GITHUB, GOOGLE, 等
  name                  String?
  lastName              String?
  isSuperAdmin          Boolean      @default(false)
  bio                   String?
  audience              Int          @default(0)
  pictureId             String?
  picture               Media?       # 個人資料圖片
  providerId            String?      # OAuth 提供者 ID
  timezone              Int
  createdAt             DateTime     @default(now())
  updatedAt             DateTime     @updatedAt
  lastReadNotifications DateTime     @default(now())
  inviteId              String?
  activated             Boolean      @default(true)
  marketplace           Boolean      @default(true)
  account               String?
  connectedAccount      Boolean      @default(false)
  lastOnline            DateTime     @default(now())
  ip                    String?
  agent                 String?      # User agent
}
```

**關聯**:
- `organizations` → 透過 UserOrganization 連接多個組織
- `picture` → Media (個人資料圖片)
- `comments` → 使用者發表的評論
- `items` → ItemUser (使用者項目/功能)
- `groupBuyer` → MessagesGroup (作為買家)
- `groupSeller` → MessagesGroup (作為賣家)
- `orderBuyer` → Orders (作為買家)
- `orderSeller` → Orders (作為賣家)
- `agencies` → SocialMediaAgency

**索引**:
- `email + providerName` (唯一)
- `lastReadNotifications`
- `inviteId`
- `account`
- `lastOnline`
- `pictureId`

---

### 2. Organization

**用途**: 代表團隊/組織 (多租戶)

**欄位**:
```prisma
model Organization {
  id           String    @id @default(uuid())
  name         String
  description  String?
  apiKey       String?   # 用於 Public API 存取
  paymentId    String?
  allowTrial   Boolean   @default(false)
  isTrailing   Boolean   @default(false)
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt
}
```

**關聯**:
- `users` → UserOrganization (成員)
- `media` → 媒體庫
- `subscription` → Subscription (一對一)
- `Integration` → 已連接的社交帳號
- `post` → 建立的貼文
- `submittedPost` → 為組織提交的貼文
- `Comments` → 貼文評論
- `notifications` → 通知
- `usedCodes` → 使用的促銷碼
- `credits` → AI 點數
- `customers` → 客戶群組
- `webhooks` → Webhook 配置
- `tags` → 貼文標籤
- `signatures` → 貼文簽名
- `autoPost` → 自動發文規則
- `sets` → 發文排程集
- `thirdParty` → 第三方整合
- `errors` → 錯誤日誌

---

### 3. UserOrganization (聯接表)

**用途**: 將使用者連接到具有角色的組織

**欄位**:
```prisma
model UserOrganization {
  id             String       @id @default(uuid())
  userId         String
  organizationId String
  disabled       Boolean      @default(false)
  role           Role         @default(USER)  # SUPERADMIN, ADMIN, USER
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
}
```

**唯一約束**: `userId + organizationId`

---

### 4. Integration

**用途**: 代表已連接的社交媒體帳號

**欄位**:
```prisma
model Integration {
  id                    String     @id @default(cuid())
  internalId            String     # 平台特定 ID
  organizationId        String
  name                  String     # 顯示名稱
  picture               String?    # 個人資料圖片 URL
  providerIdentifier    String     # 例如: "twitter", "instagram"
  type                  String     # 帳號類型
  token                 String     # OAuth 存取令牌
  disabled              Boolean    @default(false)
  tokenExpiration       DateTime?
  refreshToken          String?
  profile               String?    # JSON 個人資料資料
  postingTimes          String     @default("[{\"time\":120}, {\"time\":400}, {\"time\":700}]")
  customInstanceDetails String?    # 用於自託管平台
  customerId            String?    # 客戶分組
  rootInternalId        String?    # 父整合
  additionalSettings    String?    @default("[]")
  inBetweenSteps        Boolean    @default(false)
  refreshNeeded         Boolean    @default(false)
  deletedAt             DateTime?
  createdAt             DateTime   @default(now())
  updatedAt             DateTime?  @updatedAt
}
```

**關聯**:
- `organization` → Organization
- `posts` → 透過此整合發布的貼文
- `customer` → Customer (分組)
- `plugs` → Plugs (擴充功能)
- `webhooks` → IntegrationsWebhooks

**索引**:
- `organizationId + internalId` (唯一)
- `providerIdentifier`
- `customerId`
- `refreshNeeded`
- `disabled`

**支援的提供者**:
- X (Twitter)
- Instagram
- Facebook
- LinkedIn
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

### 5. Post

**用途**: 代表已排程或已發布的社交媒體貼文

**欄位**:
```prisma
model Post {
  id                         String    @id @default(cuid())
  state                      State     @default(QUEUE)  # QUEUE, PUBLISHED, ERROR, DRAFT
  publishDate                DateTime
  organizationId             String
  integrationId              String
  content                    String    # 貼文內容
  group                      String    # 分組識別碼
  title                      String?
  description                String?
  parentPostId               String?   # 用於串文
  releaseId                  String?
  releaseURL                 String?
  settings                   String?   # JSON 設定
  image                      String?   # 媒體 URL
  submittedForOrderId        String?   # 市集訂單
  submittedForOrganizationId String?
  approvedSubmitForOrder     APPROVED_SUBMIT_FOR_ORDER @default(NO)
  lastMessageId              String?
  intervalInDays             Int?      # 用於循環貼文
  error                      String?   # 如果失敗的錯誤訊息
  createdAt                  DateTime  @default(now())
  updatedAt                  DateTime  @updatedAt
  deletedAt                  DateTime?
}
```

**關聯**:
- `organization` → Organization
- `integration` → Integration (平台)
- `parentPost` → Post (用於串文)
- `childrenPost` → Post[] (串文回覆)
- `submittedForOrder` → Orders (市集)
- `submittedForOrganization` → Organization
- `comments` → Comments[]
- `tags` → TagsPosts[]
- `errors` → Errors[]

**索引**:
- `group`
- `publishDate`
- `state`
- `organizationId`
- `integrationId`
- `parentPostId`

**貼文狀態**:
- `QUEUE` - 已排程，等待發布
- `PUBLISHED` - 成功發布
- `ERROR` - 發布失敗
- `DRAFT` - 已儲存草稿

---

### 6. Media

**用途**: 圖片、影片和文件的檔案儲存

**欄位**:
```prisma
model Media {
  id                 String    @id @default(uuid())
  name               String
  path               String    # S3 金鑰或本地路徑
  organizationId     String
  thumbnail          String?   # 縮圖 URL
  thumbnailTimestamp Int?      # 用於影片縮圖
  alt                String?   # 無障礙替代文字
  fileSize           Int       @default(0)
  type               String    @default("image")  # image, video, pdf
  createdAt          DateTime  @default(now())
  updatedAt          DateTime  @updatedAt
  deletedAt          DateTime?
}
```

**關聯**:
- `organization` → Organization
- `userPicture` → User[] (個人資料圖片)
- `agencies` → SocialMediaAgency[]

**支援的類型**:
- 圖片: PNG, JPG, GIF, WebP
- 影片: MP4, MOV, AVI
- 文件: PDF

**索引**:
- `name`
- `organizationId`
- `type`

---

### 7. Subscription

**用途**: 管理組織帳單和方案層級

**欄位**:
```prisma
model Subscription {
  id               String           @id @default(cuid())
  organizationId   String           @unique
  subscriptionTier SubscriptionTier # STANDARD, PRO, TEAM, ULTIMATE
  identifier       String?          # Stripe 訂閱 ID
  cancelAt         DateTime?
  period           Period           # MONTHLY, YEARLY
  totalChannels    Int              # 允許的整合數量
  isLifetime       Boolean          @default(false)
  createdAt        DateTime         @default(now())
  updatedAt        DateTime         @updatedAt
  deletedAt        DateTime?
}
```

**訂閱層級**:
- `STANDARD` - 基本方案
- `PRO` - 專業方案
- `TEAM` - 團隊方案
- `ULTIMATE` - 企業方案

**期間**:
- `MONTHLY`
- `YEARLY`

---

### 8. Tags

**用途**: 分類和組織貼文

**欄位**:
```prisma
model Tags {
  id           String      @id @default(uuid())
  name         String
  color        String      # Hex 顏色碼
  orgId        String
  deletedAt    DateTime?
  createdAt    DateTime    @default(now())
  updatedAt    DateTime    @updatedAt
}
```

**關聯**:
- `organization` → Organization
- `posts` → TagsPosts[] (透過聯接表的多對多)

---

### 9. Comments

**用途**: 貼文的內部評論，用於團隊協作

**欄位**:
```prisma
model Comments {
  id             String       @id @default(uuid())
  content        String
  organizationId String
  postId         String
  userId         String
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

**關聯**:
- `organization` → Organization
- `post` → Post
- `user` → User (作者)

---

### 10. Notifications

**用途**: 使用者的應用程式內通知

**欄位**:
```prisma
model Notifications {
  id             String       @id @default(uuid())
  organizationId String
  content        String       # 通知訊息
  link           String?      # 可選連結
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

---

## 市集模型

### 11. MessagesGroup

**用途**: 買家和賣家之間的聊天串

**欄位**:
```prisma
model MessagesGroup {
  id                  String   @id @default(uuid())
  buyerOrganizationId String
  buyerId             String
  sellerId            String
  createdAt           DateTime @default(now())
  updatedAt           DateTime @updatedAt
}
```

**關聯**:
- `buyerOrganization` → Organization
- `buyer` → User
- `seller` → User
- `messages` → Messages[]
- `orders` → Orders[]

---

### 12. Orders

**用途**: 市集購買訂單

**欄位**:
```prisma
model Orders {
  id             String      @id @default(uuid())
  buyerId        String
  sellerId       String
  status         OrderStatus # PENDING, ACCEPTED, CANCELED, COMPLETED
  messageGroupId String
  captureId      String?     # 付款捕獲 ID
  createdAt      DateTime    @default(now())
  updatedAt      DateTime    @updatedAt
}
```

**訂單狀態**:
- `PENDING` - 等待接受
- `ACCEPTED` - 賣家接受
- `CANCELED` - 訂單取消
- `COMPLETED` - 訂單完成

---

### 13. OrderItems

**用途**: 訂單內的個別項目

**欄位**:
```prisma
model OrderItems {
  id            String      @id @default(uuid())
  orderId       String
  integrationId String
  quantity      Int
  price         Int         # 以分為單位
  description   String?
  createdAt     DateTime    @default(now())
  updatedAt     DateTime    @updatedAt
}
```

---

## 進階功能模型

### 14. AutoPost

**用途**: 來自 RSS 訂閱源或規則的自動發文

**欄位**:
```prisma
model AutoPost {
  id             String       @id @default(uuid())
  organizationId String
  name           String
  url            String       # RSS 訂閱源 URL
  integrations   String       # 整合 ID 的 JSON 陣列
  template       String       # 貼文模板
  enabled        Boolean      @default(true)
  lastRun        DateTime?
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

---

### 15. Webhooks

**用途**: 外發 webhook 配置

**欄位**:
```prisma
model Webhooks {
  id             String       @id @default(uuid())
  organizationId String
  url            String       # Webhook 端點
  events         String       # 事件類型的 JSON 陣列
  secret         String?      # Webhook 簽名密鑰
  enabled        Boolean      @default(true)
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

**事件類型**:
- `post.published`
- `post.failed`
- `integration.connected`
- `integration.disconnected`
- `subscription.updated`

---

### 16. ThirdParty

**用途**: 外部服務整合 (N8N, Make.com, 等)

**欄位**:
```prisma
model ThirdParty {
  id             String       @id @default(uuid())
  organizationId String
  provider       String       # n8n, make, zapier
  apiKey         String
  webhookUrl     String?
  config         String?      # JSON 配置
  enabled        Boolean      @default(true)
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

---

### 17. Signatures

**用途**: 貼文簽名/浮水印

**欄位**:
```prisma
model Signatures {
  id             String       @id @default(uuid())
  organizationId String
  content        String       # 簽名文字/HTML
  autoAdd        Boolean      # 自動附加到貼文
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

---

### 18. Sets

**用途**: 預定義發文排程

**欄位**:
```prisma
model Sets {
  id             String       @id @default(uuid())
  organizationId String
  name           String
  times          String       # 時間的 JSON 陣列
  timezone       String
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  deletedAt      DateTime?
}
```

**時間格式範例**:
```json
[
  {"day": 1, "time": "09:00"},
  {"day": 3, "time": "14:00"},
  {"day": 5, "time": "18:00"}
]
```

---

### 19. Credits

**用途**: AI 功能點數 (圖片生成等)

**欄位**:
```prisma
model Credits {
  id             String       @id @default(uuid())
  organizationId String
  credits        Int          # 剩餘點數
  type           String       @default("ai_images")
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
}
```

**點數類型**:
- `ai_images` - AI 圖片生成
- `ai_content` - AI 內容生成
- `ai_analysis` - AI 分析

---

### 20. Errors

**用途**: 錯誤日誌和偵錯

**欄位**:
```prisma
model Errors {
  id             String       @id @default(uuid())
  organizationId String
  postId         String?
  integrationId  String?
  error          String       # 錯誤訊息
  stackTrace     String?      # 完整堆疊追蹤
  context        String?      # JSON 上下文資料
  resolved       Boolean      @default(false)
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
}
```

---

### 21. Customer

**用途**: 按客戶/客戶端分組整合

**欄位**:
```prisma
model Customer {
  id           String        @id @default(uuid())
  name         String
  orgId        String
  createdAt    DateTime      @default(now())
  updatedAt    DateTime      @updatedAt
  deletedAt    DateTime?
}
```

**唯一約束**: `orgId + name + deletedAt`

---

## 列舉

### Provider
```prisma
enum Provider {
  LOCAL       # 電子郵件/密碼
  GITHUB      # GitHub OAuth
  GOOGLE      # Google OAuth
  FARCASTER   # Farcaster
  WALLET      # Web3 錢包
  GENERIC     # 通用 OAuth
}
```

### Role
```prisma
enum Role {
  SUPERADMIN  # 平台管理員
  ADMIN       # 組織管理員
  USER        # 一般使用者
}
```

### State (Post)
```prisma
enum State {
  QUEUE       # 已排程
  PUBLISHED   # 成功發布
  ERROR       # 發布失敗
  DRAFT       # 已儲存草稿
}
```

### SubscriptionTier
```prisma
enum SubscriptionTier {
  STANDARD    # 基本功能
  PRO         # 進階功能
  TEAM        # 團隊協作
  ULTIMATE    # 所有功能
}
```

### Period
```prisma
enum Period {
  MONTHLY
  YEARLY
}
```

### OrderStatus
```prisma
enum OrderStatus {
  PENDING
  ACCEPTED
  CANCELED
  COMPLETED
}
```

---

## 資料庫關聯圖

```
Organization
├── Users (透過 UserOrganization 多對多)
├── Integrations (一對多)
├── Posts (一對多)
├── Media (一對多)
├── Subscription (一對一)
├── Tags (一對多)
├── Comments (一對多)
├── Notifications (一對多)
├── Webhooks (一對多)
├── AutoPost (一對多)
├── Credits (一對多)
└── Signatures (一對多)

Integration
├── Posts (一對多)
└── Customer (多對一)

Post
├── Integration (多對一)
├── Organization (多對一)
├── Comments (一對多)
├── Tags (透過 TagsPosts 多對多)
├── ParentPost (串文的自引用)
└── ChildrenPost (串文的自引用)

User
├── Organizations (透過 UserOrganization 多對多)
├── Comments (一對多)
├── Picture (多對一 Media)
└── 市集活動 (orders, messages)
```

---

## 索引策略

### 主索引
- 所有 `id` 欄位都是帶主鍵索引的 UUID
- 多欄唯一性的複合唯一索引

### 效能索引
- 大多數表上的 `organizationId` (租戶隔離)
- `createdAt`, `updatedAt` 用於基於時間的查詢
- `deletedAt` 用於軟刪除篩選
- Posts 上的 `state`, `publishDate` (排程查詢)
- Integration 上的 `providerIdentifier` (平台篩選)

### 關聯索引
- 外鍵欄用於聯接效能
- 常見查詢模式的複合索引

---

## 軟刪除

大多數模型包含 `deletedAt` 欄位用於軟刪除:
- 允許資料恢復
- 維護引用完整性
- 在查詢中透過 `WHERE deletedAt IS NULL` 篩選

---

## 資料驗證

### Prisma 層級
- 型別檢查
- 必填欄位
- 唯一約束
- 外鍵約束

### 應用程式層級 (DTOs)
- 業務邏輯驗證
- 自訂驗證器
- 複雜規則

---

## 資料庫操作

### 常見查詢

```typescript
// 尋找組織的貼文
await prisma.post.findMany({
  where: {
    organizationId: 'org-id',
    deletedAt: null,
    state: 'QUEUE',
  },
  include: {
    integration: true,
    tags: true,
  },
  orderBy: {
    publishDate: 'asc',
  },
});

// 取得包含訂閱的組織
await prisma.organization.findUnique({
  where: { id: 'org-id' },
  include: {
    subscription: true,
    users: {
      include: {
        user: true,
      },
    },
  },
});
```

---

## 遷移策略

### 開發
```bash
pnpm prisma-db-push  # 推送模式變更
```

### 生產
- 對生產環境使用 Prisma Migrate
- 版本控制的遷移
- 回滾能力

---

## 效能考量

1. **連接池**: Prisma 管理連接池
2. **查詢最佳化**: 選擇性欄位包含
3. **急切載入**: 使用 `include` 防止 N+1 查詢
4. **分頁**: 對大型資料集實作基於游標的分頁
5. **快取**: Redis 快取經常存取的資料

---

## 下一步

- 查看 `05-api-capabilities.md` 了解 API 使用範例
- 查看 `02-backend-api.md` 了解資料存取模式
