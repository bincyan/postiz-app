# API 功能與使用指南

## 概述

本文件提供 Postiz API 功能的完整範例，包括私有驗證 API 和用於第三方整合的 Public API。

---

## 驗證

### Private API (網頁應用程式)

**方法**: 透過 cookies 進行基於 JWT 的驗證

```typescript
// 登入
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securepassword"
}

// 回應
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "John Doe"
  },
  "token": "jwt-token"  // 設定在 httpOnly cookie 中
}
```

### Public API

**方法**: Authorization 標頭中的 API 金鑰

```bash
curl -X GET https://api.postiz.com/api/v1/public/integrations \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**取得您的 API 金鑰**:
1. 登入 Postiz 儀表板
2. 導航到 Settings → API
3. 生成新的 API 金鑰
4. 複製並安全儲存

---

## 社交媒體整合管理

### 1. 列出可用平台

```bash
GET /integrations

# 回應
[
  {
    "identifier": "twitter",
    "name": "X (Twitter)",
    "icon": "https://...",
    "supportsAuth": true,
    "features": ["posts", "media", "analytics"]
  },
  {
    "identifier": "instagram",
    "name": "Instagram",
    "icon": "https://...",
    "supportsAuth": true,
    "features": ["posts", "media", "stories"]
  }
  // ... 更多平台
]
```

### 2. 連接社交帳號 (OAuth 流程)

```typescript
// 步驟 1: 啟動 OAuth
POST /integrations/connect
{
  "providerIdentifier": "twitter"
}

// 回應
{
  "authUrl": "https://twitter.com/oauth/authorize?..."
}

// 步驟 2: 使用者在平台上授權

// 步驟 3: OAuth 回呼 (自動)
GET /auth/callback/twitter?code=...

// 步驟 4: 建立整合
{
  "id": "integration-uuid",
  "name": "John's Twitter",
  "providerIdentifier": "twitter",
  "picture": "https://...",
  "connected": true
}
```

### 3. 列出已連接的帳號

```bash
GET /integrations/list

# 回應
{
  "integrations": [
    {
      "id": "uuid-1",
      "name": "Personal Twitter",
      "providerIdentifier": "twitter",
      "picture": "https://...",
      "disabled": false,
      "refreshNeeded": false,
      "customer": {
        "id": "customer-uuid",
        "name": "Personal Accounts"
      }
    },
    {
      "id": "uuid-2",
      "name": "Brand Instagram",
      "providerIdentifier": "instagram",
      "picture": "https://...",
      "disabled": false
    }
  ]
}
```

### 4. 按客戶分組整合

```bash
PUT /integrations/{integrationId}/customer-name
{
  "customerName": "Client A Accounts"
}

# 回應
{
  "success": true,
  "integration": {
    "id": "uuid",
    "customer": {
      "name": "Client A Accounts"
    }
  }
}
```

### 5. 斷開帳號

```bash
DELETE /integrations/{integrationId}

# 回應
{
  "success": true,
  "message": "Integration disconnected"
}
```

---

## 貼文建立與排程

### 1. 建立簡單貼文

```typescript
POST /posts
Content-Type: application/json

{
  "content": "Hello World! 🌍",
  "integrations": [
    "twitter-integration-id",
    "linkedin-integration-id"
  ],
  "publishDate": "2025-01-15T10:00:00Z",
  "state": "QUEUE"  // 或 "DRAFT"
}

// 回應
{
  "id": "post-uuid",
  "content": "Hello World! 🌍",
  "state": "QUEUE",
  "publishDate": "2025-01-15T10:00:00Z",
  "integrations": [
    {
      "id": "twitter-integration-id",
      "name": "Twitter Account",
      "providerIdentifier": "twitter"
    },
    {
      "id": "linkedin-integration-id",
      "name": "LinkedIn Profile",
      "providerIdentifier": "linkedin"
    }
  ]
}
```

### 2. 建立帶媒體的貼文

```typescript
// 步驟 1: 上傳媒體
POST /media/upload
Content-Type: multipart/form-data

FormData: {
  file: [image file]
}

// 回應
{
  "id": "media-uuid",
  "name": "image.jpg",
  "path": "https://...",
  "thumbnail": "https://...",
  "type": "image"
}

// 步驟 2: 建立帶媒體的貼文
POST /posts
{
  "content": "Check out this image!",
  "integrations": ["twitter-id"],
  "publishDate": "2025-01-15T14:00:00Z",
  "image": "https://media-url",  // 或陣列以包含多張圖片
  "settings": {
    "media": ["media-uuid"]
  }
}
```

### 3. 建立帶平台特定設定的貼文

```typescript
POST /posts
{
  "content": "Platform-specific post",
  "integrations": ["twitter-id", "instagram-id"],
  "publishDate": "2025-01-15T16:00:00Z",
  "settings": {
    "twitter": {
      "replyToTweetId": "tweet-id"  // 建立串文
    },
    "instagram": {
      "location": {
        "name": "New York, NY",
        "id": "location-id"
      },
      "altText": "Image description"
    }
  }
}
```

### 4. 建立串文/多貼文

```typescript
// 建立父貼文
POST /posts
{
  "content": "Thread 1/3: Introduction",
  "integrations": ["twitter-id"],
  "publishDate": "2025-01-15T10:00:00Z"
}

// 回應: { "id": "parent-post-id" }

// 建立子貼文
POST /posts
{
  "content": "Thread 2/3: Details",
  "integrations": ["twitter-id"],
  "publishDate": "2025-01-15T10:01:00Z",
  "parentPostId": "parent-post-id"
}

POST /posts
{
  "content": "Thread 3/3: Conclusion",
  "integrations": ["twitter-id"],
  "publishDate": "2025-01-15T10:02:00Z",
  "parentPostId": "parent-post-id"
}
```

### 5. 建立循環貼文

```typescript
POST /posts
{
  "content": "Daily motivation quote",
  "integrations": ["twitter-id"],
  "publishDate": "2025-01-15T09:00:00Z",
  "intervalInDays": 1  // 每天重複
}
```

### 6. 儲存為草稿

```typescript
POST /posts
{
  "content": "Draft post content",
  "integrations": ["twitter-id"],
  "state": "DRAFT",
  "publishDate": "2025-01-20T10:00:00Z"
}
```

### 7. AI 驅動的貼文生成

```typescript
POST /posts/generate
{
  "prompt": "Write a post about the benefits of social media scheduling",
  "tone": "professional",
  "platform": "linkedin",
  "length": "medium"
}

// 回應
{
  "suggestions": [
    {
      "content": "Social media scheduling is a game-changer...",
      "hashtags": ["#SocialMedia", "#Productivity"],
      "emoji": true
    },
    {
      "content": "Maximize your social media impact...",
      "hashtags": ["#Marketing", "#Automation"],
      "emoji": false
    }
  ]
}
```

---

## 貼文管理

### 1. 列出貼文

```bash
GET /posts?state=QUEUE&limit=20&offset=0

# 查詢參數:
# - state: QUEUE, PUBLISHED, ERROR, DRAFT
# - integrationId: 按平台篩選
# - startDate: ISO 日期
# - endDate: ISO 日期
# - limit: 每頁結果數
# - offset: 分頁偏移
# - group: 群組識別碼

# 回應
{
  "posts": [
    {
      "id": "post-uuid",
      "content": "...",
      "state": "QUEUE",
      "publishDate": "2025-01-15T10:00:00Z",
      "integration": {...},
      "tags": [...]
    }
  ],
  "total": 150,
  "hasMore": true
}
```

### 2. 取得貼文詳情

```bash
GET /posts/{postId}

# 回應
{
  "id": "post-uuid",
  "content": "...",
  "state": "PUBLISHED",
  "publishDate": "2025-01-15T10:00:00Z",
  "integration": {
    "id": "integration-uuid",
    "name": "Twitter Account",
    "providerIdentifier": "twitter"
  },
  "comments": [
    {
      "id": "comment-uuid",
      "content": "Great post!",
      "user": {
        "name": "Team Member"
      },
      "createdAt": "2025-01-14T15:30:00Z"
    }
  ],
  "tags": [
    {
      "name": "Campaign 2025",
      "color": "#FF5733"
    }
  ]
}
```

### 3. 更新貼文

```bash
PUT /posts/{postId}
{
  "content": "Updated content",
  "publishDate": "2025-01-16T10:00:00Z"
}
```

### 4. 刪除貼文

```bash
DELETE /posts/{postId}

# 回應
{
  "success": true,
  "message": "Post deleted"
}
```

### 5. 為貼文新增評論

```bash
POST /posts/{postId}/comments
{
  "comment": "Please review this before publishing"
}

# 回應
{
  "id": "comment-uuid",
  "content": "Please review this before publishing",
  "user": {...},
  "createdAt": "2025-01-14T16:00:00Z"
}
```

---

## 標籤管理

### 1. 建立標籤

```bash
POST /posts/tags
{
  "name": "Q1 Campaign",
  "color": "#3498db"
}

# 回應
{
  "id": "tag-uuid",
  "name": "Q1 Campaign",
  "color": "#3498db"
}
```

### 2. 列出標籤

```bash
GET /posts/tags

# 回應
{
  "tags": [
    {
      "id": "tag-uuid-1",
      "name": "Q1 Campaign",
      "color": "#3498db"
    },
    {
      "id": "tag-uuid-2",
      "name": "Product Launch",
      "color": "#e74c3c"
    }
  ]
}
```

### 3. 標記貼文

```bash
PUT /posts/{postId}/tags
{
  "tags": ["tag-uuid-1", "tag-uuid-2"]
}
```

---

## 分析與見解

### 1. 取得概覽分析

```bash
GET /analytics/overview?startDate=2025-01-01&endDate=2025-01-31

# 回應
{
  "period": {
    "start": "2025-01-01",
    "end": "2025-01-31"
  },
  "totalPosts": 120,
  "totalImpressions": 45000,
  "totalEngagements": 3500,
  "engagementRate": 7.78,
  "followerGrowth": 250,
  "topPerformingPosts": [
    {
      "postId": "uuid",
      "content": "...",
      "impressions": 5000,
      "engagements": 450
    }
  ],
  "platformBreakdown": {
    "twitter": {
      "posts": 40,
      "impressions": 15000,
      "engagements": 1200
    },
    "linkedin": {
      "posts": 30,
      "impressions": 18000,
      "engagements": 1500
    }
  }
}
```

### 2. 取得平台特定分析

```bash
GET /analytics/integration/{integrationId}?period=30d

# 回應
{
  "integration": {
    "id": "uuid",
    "name": "Twitter Account",
    "providerIdentifier": "twitter"
  },
  "metrics": {
    "followers": 5420,
    "followerGrowth": 150,
    "impressions": 25000,
    "engagements": 2000,
    "clicks": 800,
    "engagementRate": 8.0
  },
  "posts": [
    {
      "id": "post-uuid",
      "content": "...",
      "publishDate": "2025-01-15T10:00:00Z",
      "metrics": {
        "impressions": 1500,
        "likes": 45,
        "retweets": 12,
        "replies": 8
      }
    }
  ],
  "trends": {
    "impressions": [
      {"date": "2025-01-01", "value": 800},
      {"date": "2025-01-02", "value": 950}
    ]
  }
}
```

### 3. 取得貼文統計

```bash
GET /posts/{postId}/statistics

# 回應
{
  "postId": "uuid",
  "platform": "twitter",
  "metrics": {
    "impressions": 2500,
    "likes": 85,
    "retweets": 23,
    "replies": 12,
    "clicks": 45,
    "engagementRate": 6.6
  },
  "demographics": {
    "topCountries": [
      {"country": "US", "percentage": 45},
      {"country": "UK", "percentage": 20}
    ],
    "ageGroups": [
      {"range": "25-34", "percentage": 35}
    ]
  }
}
```

---

## 媒體庫

### 1. 上傳媒體

```bash
POST /media/upload
Content-Type: multipart/form-data

FormData: {
  file: [file],
  alt: "Image description"
}

# 回應
{
  "id": "media-uuid",
  "name": "vacation.jpg",
  "path": "https://cdn.postiz.com/...",
  "thumbnail": "https://cdn.postiz.com/thumb-...",
  "type": "image",
  "fileSize": 245678,
  "alt": "Image description"
}
```

### 2. 列出媒體

```bash
GET /media?type=image&limit=20&offset=0

# 回應
{
  "media": [
    {
      "id": "uuid-1",
      "name": "image1.jpg",
      "path": "https://...",
      "thumbnail": "https://...",
      "type": "image",
      "fileSize": 245678,
      "createdAt": "2025-01-10T10:00:00Z"
    }
  ],
  "total": 85,
  "hasMore": true
}
```

### 3. 刪除媒體

```bash
DELETE /media/{mediaId}

# 回應
{
  "success": true
}
```

---

## 團隊協作

### 1. 邀請團隊成員

```bash
POST /agencies/{orgId}/invite
{
  "email": "newmember@example.com",
  "role": "USER"  // ADMIN 或 USER
}

# 回應
{
  "success": true,
  "invitation": {
    "email": "newmember@example.com",
    "role": "USER",
    "inviteLink": "https://..."
  }
}
```

### 2. 列出團隊成員

```bash
GET /agencies/{orgId}/members

# 回應
{
  "members": [
    {
      "userId": "uuid",
      "name": "John Doe",
      "email": "john@example.com",
      "role": "ADMIN",
      "joinedAt": "2024-01-01T00:00:00Z"
    }
  ]
}
```

### 3. 更新成員角色

```bash
PUT /agencies/{orgId}/members/{userId}
{
  "role": "ADMIN"
}
```

### 4. 移除成員

```bash
DELETE /agencies/{orgId}/members/{userId}
```

---

## Webhooks

### 1. 建立 Webhook

```bash
POST /webhooks
{
  "url": "https://your-server.com/webhook",
  "events": ["post.published", "post.failed", "integration.connected"],
  "secret": "your-webhook-secret"
}

# 回應
{
  "id": "webhook-uuid",
  "url": "https://your-server.com/webhook",
  "events": ["post.published", "post.failed", "integration.connected"],
  "enabled": true
}
```

### 2. Webhook 事件格式

當事件發生時，Postiz 發送:

```json
POST https://your-server.com/webhook
X-Postiz-Signature: sha256=...
Content-Type: application/json

{
  "event": "post.published",
  "timestamp": "2025-01-15T10:00:00Z",
  "data": {
    "postId": "uuid",
    "integrationId": "uuid",
    "platform": "twitter",
    "content": "...",
    "publishedUrl": "https://twitter.com/..."
  }
}
```

### 3. 驗證 Webhook 簽名

```typescript
import crypto from 'crypto';

function verifyWebhook(payload: string, signature: string, secret: string): boolean {
  const hmac = crypto.createHmac('sha256', secret);
  const digest = 'sha256=' + hmac.update(payload).digest('hex');
  return crypto.timingSafeEqual(Buffer.from(signature), Buffer.from(digest));
}
```

---

## 自動化

### 1. 建立 RSS 自動發文

```bash
POST /autopost
{
  "name": "Blog Auto-Post",
  "url": "https://blog.example.com/feed.xml",
  "integrations": ["twitter-id", "linkedin-id"],
  "template": "New blog post: {title} - {link}",
  "enabled": true
}

# 回應
{
  "id": "autopost-uuid",
  "name": "Blog Auto-Post",
  "url": "https://blog.example.com/feed.xml",
  "lastRun": null,
  "enabled": true
}
```

### 2. 建立發文排程集

```bash
POST /sets
{
  "name": "Weekday Morning Posts",
  "times": [
    {"day": 1, "time": "09:00"},
    {"day": 2, "time": "09:00"},
    {"day": 3, "time": "09:00"},
    {"day": 4, "time": "09:00"},
    {"day": 5, "time": "09:00"}
  ],
  "timezone": "America/New_York"
}
```

---

## Public API 範例

### 使用 Node.js SDK

```bash
npm install @postiz/node
```

```typescript
import { Postiz } from '@postiz/node';

const client = new Postiz({
  apiKey: 'your-api-key'
});

// 列出整合
const integrations = await client.integrations.list();

// 建立貼文
const post = await client.posts.create({
  content: 'Hello from API!',
  integrations: ['integration-id'],
  publishDate: new Date('2025-01-15T10:00:00Z')
});

// 取得分析
const analytics = await client.analytics.overview({
  startDate: '2025-01-01',
  endDate: '2025-01-31'
});
```

### 使用 cURL

```bash
# 建立貼文
curl -X POST https://api.postiz.com/api/v1/public/integrations/post \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Hello from cURL!",
    "integrationId": "integration-uuid",
    "publishDate": "2025-01-15T10:00:00Z"
  }'
```

---

## 速率限制

### Private API
- 已驗證使用者: 1000 次請求/小時
- 依訂閱層級而異

### Public API
- Standard: 100 次請求/分鐘
- Pro: 500 次請求/分鐘
- Team: 1000 次請求/分鐘
- Ultimate: 5000 次請求/分鐘

### 速率限制標頭
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1705320000
```

---

## 錯誤處理

### 錯誤回應格式

```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "content",
      "message": "Content is required"
    },
    {
      "field": "publishDate",
      "message": "Publish date must be in the future"
    }
  ]
}
```

### 常見錯誤代碼

- `400` - Bad Request (驗證錯誤)
- `401` - Unauthorized (無效的 API 金鑰)
- `403` - Forbidden (權限不足)
- `404` - Not Found
- `429` - Too Many Requests (速率限制)
- `500` - Internal Server Error

---

## 最佳實踐

### 1. 批次操作
不要逐個建立貼文，而是批次處理:

```typescript
const posts = [/* 貼文資料陣列 */];
const results = await Promise.all(
  posts.map(post => client.posts.create(post))
);
```

### 2. 錯誤處理
始終優雅地處理 API 錯誤:

```typescript
try {
  const post = await client.posts.create(data);
} catch (error) {
  if (error.statusCode === 429) {
    // 速率限制 - 等待並重試
    await sleep(60000);
    return retry();
  }
  // 記錄錯誤並通知使用者
  console.error(error);
}
```

### 3. 使用 Webhooks 進行即時更新
不要輪詢貼文狀態，而是:

```typescript
// 設定 webhook 以接收發布通知
await client.webhooks.create({
  url: 'https://your-app.com/webhook',
  events: ['post.published', 'post.failed']
});
```

### 4. 最佳化媒體上傳
- 上傳前壓縮圖片
- 使用適當的格式 (網頁使用 WebP)
- 如果可能，在客戶端生成縮圖

---

## 整合範例

### N8N 工作流程

Postiz 與 N8N 整合以實現進階自動化:

```json
{
  "nodes": [
    {
      "type": "n8n-nodes-postiz",
      "operation": "createPost",
      "parameters": {
        "content": "={{$json.content}}",
        "integrations": ["twitter-id"],
        "publishDate": "={{$json.scheduledTime}}"
      }
    }
  ]
}
```

### Make.com (Integromat)

可作為 Make.com 模組用於視覺化自動化。

### Zapier

透過 Zapier 整合連接 Postiz 與 5000+ 個應用程式。

---

## 使用案例

### 1. 內容策展機器人
```typescript
// 獲取熱門內容
const trends = await fetchTrends();

// 使用 AI 生成貼文
const suggestions = await client.posts.generate({
  prompt: `Create posts about: ${trends.join(', ')}`,
  count: 5
});

// 排程貼文
for (const suggestion of suggestions) {
  await client.posts.create({
    content: suggestion.content,
    integrations: allIntegrations,
    publishDate: getNextSlot()
  });
}
```

### 2. 跨平台活動
```typescript
// 在所有平台上建立活動
const platforms = ['twitter', 'linkedin', 'facebook', 'instagram'];

for (const platform of platforms) {
  const integration = integrations.find(i => i.providerIdentifier === platform);

  await client.posts.create({
    content: customizeForPlatform(content, platform),
    integrations: [integration.id],
    publishDate: campaignStartDate,
    tags: ['campaign-2025']
  });
}
```

### 3. 分析儀表板
```typescript
// 獲取所有平台的分析
const analytics = await Promise.all(
  integrations.map(integration =>
    client.analytics.integration(integration.id, { period: '30d' })
  )
);

// 生成報告
const report = generateReport(analytics);
sendEmailReport(report);
```

---

## 支援與資源

- **API 文件**: https://docs.postiz.com/public-api
- **NPM 上的 SDK**: https://www.npmjs.com/package/@postiz/node
- **N8N Node**: https://www.npmjs.com/package/n8n-nodes-postiz
- **社群 Discord**: https://discord.postiz.com

---

## 結論

Postiz API 提供全面的功能:
- ✅ 多平台社交媒體管理
- ✅ 自動化內容排程
- ✅ 團隊協作
- ✅ 分析和見解
- ✅ AI 驅動的內容生成
- ✅ Webhook 整合
- ✅ 第三方自動化

無論您是建置自訂儀表板、自動化工作流程還是與其他服務整合，Postiz API 都能滿足您的需求。
