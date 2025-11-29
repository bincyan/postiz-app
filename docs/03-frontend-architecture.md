# Frontend 架構

## 概述

Postiz 前端使用 **Next.js 14+** 配合 App Router 建構，提供具有出色效能和 SEO 的現代化伺服器渲染 React 應用程式。

## 技術堆疊

### 核心框架
- **Next.js**: 14.2.30 (App Router)
- **React**: 18.3.1
- **TypeScript**: 5.5.4
- **Node.js**: 22.12.0+

### UI 函式庫
- **Mantine**: 5.10.5 - 元件函式庫
- **TailwindCSS**: 4.1.7 - Utility-first CSS
- **@tailwindcss/postcss**: 用於 Next.js 整合
- **tailwind-scrollbar**: 自訂捲軸
- **tailwindcss-rtl**: RTL 語言支援

### 狀態管理
- **Zustand**: 5.0.5 - 輕量級狀態管理
- **SWR**: 2.2.5 - 資料獲取和快取
- **React Hook Form**: 7.58.1 - 表單狀態管理

### 富文本編輯
- **TipTap**: 3.0.6 - WYSIWYG 編輯器
  - 粗體、底線、標題
  - 連結和提及
  - 自訂擴充功能
- **@uiw/react-md-editor**: Markdown 編輯器
- **Emoji Picker React**: 表情符號選擇

### 檔案上傳
- **Uppy**: 4.4.6 - 現代化檔案上傳器
  - 儀表板 UI
  - 拖放功能
  - 進度追蹤
  - AWS S3 整合
  - 壓縮支援

### 資料視覺化
- **Chart.js**: 4.4.1 - 分析圖表
- **React Country Flag**: 國家指標

### AI 整合
- **CopilotKit**: 1.10.6 - AI 助理 UI
  - Textarea 整合
  - 聊天介面
  - Runtime 整合

### 工具
- **dayjs**: 日期操作
- **lodash**: 工具函數
- **clsx**: 類別名稱組合
- **i18next**: 國際化
- **react-i18next**: i18n 的 React 綁定
- **accept-language**: 語言偵測

### 監控
- **Sentry**: 錯誤追蹤和效能
  - @sentry/nextjs
  - @sentry/react
- **PostHog**: 產品分析

---

## 應用程式結構

```
apps/frontend/src/
├── app/                          # Next.js App Router
│   ├── (app)/                    # 已驗證應用程式群組
│   │   ├── (site)/              # 主應用程式路由
│   │   │   ├── launches/        # 貼文排程
│   │   │   ├── analytics/       # 分析儀表板
│   │   │   ├── integrations/    # 社交帳號管理
│   │   │   ├── media/           # 媒體庫
│   │   │   ├── settings/        # 使用者/組織設定
│   │   │   ├── billing/         # 訂閱管理
│   │   │   ├── agents/          # AI 代理
│   │   │   ├── plugs/           # 插件管理
│   │   │   └── third-party/     # 第三方整合
│   │   ├── (preview)/           # 預覽頁面
│   │   ├── auth/                # 驗證頁面
│   │   └── api/                 # API 路由 (Next.js)
│   ├── (extension)/             # 瀏覽器擴充功能彈出視窗
│   ├── layout.tsx               # 根佈局
│   ├── page.tsx                 # 登陸頁面
│   └── globals.css              # 全域樣式
├── components/                   # React 元件
│   ├── launches/                # 貼文建立與排程
│   ├── analytics/               # 分析元件
│   ├── marketplace/             # 市集 UI
│   ├── settings/                # 設定表單
│   ├── layout/                  # 佈局元件
│   ├── ui/                      # 可重用 UI 元件
│   └── ...                      # 功能特定元件
├── middleware.ts                # Next.js 中介軟體
└── instrumentation.ts           # 監控設定
```

---

## 路由架構

### App Router (Next.js 14+)

#### 路由群組

**`(app)` 群組** - 已驗證路由
- 需要使用者登入
- 共享共同佈局
- 組織範圍

**`(site)` 群組** - 主應用程式
- 儀表板和功能
- 嵌套在 (app) 下

**`(extension)` 群組** - 瀏覽器擴充功能
- 彈出視窗的簡化 UI
- 有限功能

### 主要路由

#### 1. Launches (`/launches`)
**用途**: 貼文建立和排程

**功能**:
- 多平台貼文編輯器
- 富文本編輯
- 媒體附件
- 排程行事曆
- 草稿管理
- AI 驅動的建議

**頁面**:
- `/launches` - 貼文清單/行事曆
- `/launches/new` - 建立新貼文
- `/launches/:id` - 編輯貼文
- `/launches/:id/preview` - 預覽貼文

#### 2. Analytics (`/analytics`)
**用途**: 效能指標和見解

**功能**:
- 概覽儀表板
- 平台特定分析
- 互動指標
- 成長趨勢
- 匯出資料

**頁面**:
- `/analytics` - 概覽
- `/analytics/:integrationId` - 平台分析

#### 3. Integrations (`/integrations`)
**用途**: 社交媒體帳號管理

**功能**:
- 連接帳號
- OAuth 流程
- 帳號分組
- 令牌刷新
- 斷開帳號

**頁面**:
- `/integrations` - 已連接帳號清單
- `/integrations/connect` - 新增帳號

#### 4. Media (`/media`)
**用途**: 媒體庫管理

**功能**:
- 上傳檔案
- 瀏覽庫
- 搜尋和篩選
- 刪除媒體
- 組織資料夾

#### 5. Settings (`/settings`)
**用途**: 使用者和組織配置

**頁面**:
- `/settings/profile` - 使用者個人資料
- `/settings/organization` - 組織設定
- `/settings/team` - 團隊管理
- `/settings/notifications` - 通知偏好設定
- `/settings/api` - API 金鑰管理
- `/settings/billing` - 訂閱管理

#### 6. Billing (`/billing`)
**用途**: 訂閱和付款管理

**功能**:
- 目前方案詳情
- 升級/降級
- 付款方式
- 發票歷史
- 使用追蹤

#### 7. Agents (`/agents`)
**用途**: AI 代理配置

**功能**:
- AI 工作流程管理
- 代理訓練
- 自動化規則

#### 8. Third Party (`/third-party`)
**用途**: 外部整合

**功能**:
- N8N 工作流程
- Make.com 整合
- Zapier 連接
- 自訂 webhooks

---

## 元件架構

### 元件組織

```
components/
├── launches/                     # 貼文建立
│   ├── calendar-view/           # 行事曆 UI
│   ├── post-editor/             # 富文本編輯器
│   ├── media-uploader/          # 檔案上傳
│   ├── platform-selector/       # 平台選擇器
│   └── scheduling/              # 日期/時間選擇器
├── analytics/
│   ├── overview-chart/          # 主圖表
│   ├── metrics-card/            # 統計卡片
│   └── platform-breakdown/      # 每個平台的統計
├── marketplace/
│   ├── post-browser/            # 瀏覽貼文
│   ├── offer-form/              # 出價
│   └── transaction-list/        # 購買歷史
├── settings/
│   ├── profile-form/            # 使用者個人資料
│   ├── team-management/         # 邀請成員
│   └── api-keys/                # API 金鑰管理
├── layout/
│   ├── sidebar/                 # 導航側邊欄
│   ├── topbar/                  # 標頭
│   ├── modal/                   # 模態對話框
│   └── loading/                 # 載入狀態
└── ui/                          # 可重用元件
    ├── button/
    ├── input/
    ├── select/
    ├── modal/
    ├── tooltip/
    └── ...
```

### 元件模式

#### 1. Server Components (預設)
用於:
- 靜態內容
- 資料獲取
- SEO 關鍵頁面

```typescript
// Server Component
export default async function AnalyticsPage() {
  const data = await fetchAnalytics();
  return <AnalyticsChart data={data} />;
}
```

#### 2. Client Components
用於:
- 互動性
- 瀏覽器 API
- 狀態管理

```typescript
'use client';

export function PostEditor() {
  const [content, setContent] = useState('');
  // ... 互動邏輯
}
```

#### 3. 混合方法
Server component 包裝 client components:

```typescript
// Server Component
export default async function LaunchesPage() {
  const initialPosts = await getPosts();

  return (
    <div>
      <PostList initialData={initialPosts} /> {/* Client */}
    </div>
  );
}
```

---

## 狀態管理

### 1. Zustand (全域狀態)

用於:
- 使用者會話
- UI 狀態 (側邊欄、模態框)
- 臨時資料

```typescript
// 範例 store
import { create } from 'zustand';

interface UserStore {
  user: User | null;
  setUser: (user: User) => void;
  logout: () => void;
}

export const useUserStore = create<UserStore>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  logout: () => set({ user: null }),
}));
```

### 2. SWR (伺服器狀態)

用於:
- API 資料獲取
- 快取
- 重新驗證
- 樂觀更新

```typescript
import useSWR from 'swr';

function PostList() {
  const { data, error, mutate } = useSWR('/api/posts', fetcher);

  if (error) return <Error />;
  if (!data) return <Loading />;

  return <PostGrid posts={data} />;
}
```

### 3. React Hook Form (表單狀態)

用於:
- 表單管理
- 驗證
- 錯誤處理

```typescript
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';

function CreatePostForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: yupResolver(postSchema),
  });

  const onSubmit = async (data) => {
    await createPost(data);
  };

  return <form onSubmit={handleSubmit(onSubmit)}>...</form>;
}
```

---

## 主要功能實作

### 1. 貼文編輯器

**技術**:
- TipTap (富文本)
- React Hook Form (狀態)
- Uppy (媒體上傳)

**功能**:
- 多平台支援
- 每個平台的字元計數
- 媒體預覽
- 表情符號選擇器
- Hashtag 建議
- 提及自動完成
- AI 內容生成

**字元限制**:
```typescript
const PLATFORM_LIMITS = {
  twitter: 280,
  linkedin: 3000,
  instagram: 2200,
  facebook: 63206,
  // ... 等
};
```

### 2. 行事曆檢視

**技術**:
- 自訂行事曆元件
- 拖放 (react-dnd)
- 日期處理 (dayjs)

**功能**:
- 月/週/日檢視
- 拖曳以重新排程
- 多選
- 快速篩選
- 依平台顏色編碼

### 3. 分析儀表板

**技術**:
- Chart.js
- SWR 用於資料獲取
- 日期範圍選擇器

**顯示的指標**:
- 曝光次數
- 互動率
- 追蹤者成長
- 熱門貼文
- 平台比較

### 4. 媒體庫

**技術**:
- Uppy Dashboard
- AWS S3 上傳
- 圖片最佳化

**功能**:
- 拖放上傳
- 多檔案選擇
- 進度追蹤
- 搜尋和篩選
- 資料夾組織

### 5. AI 助理 (Copilot)

**技術**:
- CopilotKit
- OpenAI 整合
- 串流回應

**功能**:
- 內容生成
- 貼文改進
- Hashtag 建議
- 語調調整
- 翻譯

---

## 樣式系統

### TailwindCSS 配置

```javascript
// tailwind.config.js
module.exports = {
  content: ['./src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: '#FF6B6B',
        secondary: '#4ECDC4',
        // ... 自訂顏色
      },
    },
  },
  plugins: [
    require('tailwind-scrollbar'),
    require('tailwindcss-rtl'),
  ],
};
```

### Mantine 主題

一致元件樣式的自訂主題配置:

```typescript
const theme = {
  colorScheme: 'dark',
  primaryColor: 'brand',
  fontFamily: 'Inter, sans-serif',
  // ... 自訂主題
};
```

### 深色模式支援

內建深色模式，帶系統偏好設定偵測。

---

## 國際化 (i18n)

### 支援的語言
- English (預設)
- Spanish
- French
- German
- Chinese
- Japanese
- 以及更多...

### 實作

```typescript
import { useTranslation } from 'react-i18next';

function Component() {
  const { t } = useTranslation();

  return <h1>{t('welcome.title')}</h1>;
}
```

### 語言檔案

```
public/locales/
├── en/
│   └── common.json
├── es/
│   └── common.json
└── ...
```

---

## 驗證流程

### 1. 登入
```
使用者輸入憑證
  ↓
POST /api/auth/login
  ↓
接收 JWT 令牌
  ↓
儲存在 httpOnly cookie
  ↓
重新導向到 /launches
```

### 2. 受保護路由

**中介軟體** (`middleware.ts`):
- 檢查 JWT 令牌
- 驗證使用者會話
- 如果未驗證則重新導向到 /auth

### 3. OAuth 流程

```
點擊 "Connect Twitter"
  ↓
重新導向到 Twitter OAuth
  ↓
使用者批准
  ↓
回呼到 /auth/callback/twitter
  ↓
儲存令牌
  ↓
重新導向回 /integrations
```

---

## API 整合

### Fetch Helper

```typescript
// 自訂 fetch 包裝器
async function apiFetch(endpoint: string, options?: RequestInit) {
  const response = await fetch(`${API_URL}${endpoint}`, {
    ...options,
    credentials: 'include', // 包含 cookies
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
  });

  if (!response.ok) throw new Error(response.statusText);
  return response.json();
}
```

### SWR 整合

```typescript
const fetcher = (url: string) => apiFetch(url);

function usePosts() {
  return useSWR('/posts', fetcher);
}
```

---

## 效能最佳化

### 1. 程式碼分割
- 基於路由的分割 (自動)
- 重型元件的動態匯入
- 延遲載入媒體

### 2. 圖片最佳化
- Next.js Image 元件
- 自動格式選擇 (WebP)
- 響應式圖片
- 延遲載入

### 3. 快取策略
- SWR 快取
- 瀏覽器快取標頭
- Service worker (可選)

### 4. 打包最佳化
- Tree shaking
- 最小化
- 壓縮 (gzip/brotli)

---

## 錯誤處理

### 錯誤邊界

```typescript
'use client';

export function ErrorBoundary({ children }) {
  return (
    <Sentry.ErrorBoundary fallback={<ErrorUI />}>
      {children}
    </Sentry.ErrorBoundary>
  );
}
```

### Toast 通知

```typescript
import { showNotification } from '@mantine/notifications';

showNotification({
  title: 'Success',
  message: 'Post created successfully',
  color: 'green',
});
```

---

## 測試

### 測試堆疊
- Jest
- React Testing Library
- Playwright (E2E)

### 元件測試

```typescript
import { render, screen } from '@testing-library/react';

describe('PostEditor', () => {
  it('renders editor', () => {
    render(<PostEditor />);
    expect(screen.getByRole('textbox')).toBeInTheDocument();
  });
});
```

---

## 建置與部署

### 建置流程

```bash
# 開發
pnpm dev:frontend

# 生產建置
pnpm build:frontend

# 啟動生產伺服器
pnpm start:prod:frontend
```

### 環境變數

```env
NEXT_PUBLIC_API_URL=http://localhost:3000
NEXT_PUBLIC_SENTRY_DSN=...
NEXT_PUBLIC_POSTHOG_KEY=...
```

### Docker 部署

針對生產環境最佳化的多階段 Dockerfile。

---

## 瀏覽器相容性

- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

---

## 無障礙性

- ARIA 標籤
- 鍵盤導航
- 螢幕閱讀器支援
- 焦點管理
- 顏色對比度合規

---

## 下一步

- 查看 `04-database-models.md` 了解資料結構
- 查看 `05-api-capabilities.md` 了解 API 整合範例
